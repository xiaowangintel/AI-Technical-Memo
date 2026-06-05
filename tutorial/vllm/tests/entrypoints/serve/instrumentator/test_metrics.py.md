# test_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/serve/instrumentator/test_metrics.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers serve subsystem behavior. The file defines 4 test(s), 4 fixture(s), and 4 helper/class block(s) to validate this area. / [CN] 该文件覆盖服务子系统行为。它定义了 4 个测试、4 个 fixture，以及 4 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L20)
```python
import asyncio
import subprocess
import sys
import tempfile
import time
from http import HTTPStatus

import openai
import pytest
import pytest_asyncio
import requests
from prometheus_client.parser import text_string_to_metric_families
from transformers import AutoTokenizer

from tests.conftest import LocalAssetServer
from tests.utils import RemoteOpenAIServer
from vllm import version
from vllm.utils.network_utils import get_open_port
```
**EN:** Imports standard-library modules such as `asyncio`, `http.HTTPStatus`, `subprocess`, third-party packages like `openai`, `prometheus_client.parser.text_string_to_metric_families`, `pytest`, project helpers such as `tests.conftest.LocalAssetServer`, `tests.utils.RemoteOpenAIServer`, `vllm.utils.network_utils.get_open_port`.
**CN:** 导入标准库模块（如 `asyncio`、`http.HTTPStatus`、`subprocess`）、第三方包（如 `openai`、`prometheus_client.parser.text_string_to_metric_families`、`pytest`）、项目内辅助模块（如 `tests.conftest.LocalAssetServer`、`tests.utils.RemoteOpenAIServer`、`vllm.utils.network_utils.get_open_port`）。

### Module setup / 模块级配置: MODELS, PREV_MINOR_VERSION (L22-L26)
```python
MODELS = {
    "text": "TinyLlama/TinyLlama-1.1B-Chat-v1.0",
    "multimodal": "HuggingFaceTB/SmolVLM-256M-Instruct",
}
PREV_MINOR_VERSION = version._prev_minor_version()
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODELS`, `PREV_MINOR_VERSION`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODELS`、`PREV_MINOR_VERSION`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: model_key (L29-L31)
```python
@pytest.fixture(scope="module", params=list(MODELS.keys()))
def model_key(request):
    yield request.param
```
**EN:** This fixture prepares `model_key` for dependent tests. Key inputs are `request`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `model_key`。 关键输入包括 `request`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: default_server_args (L34-L45)
```python
@pytest.fixture(scope="module")
def default_server_args():
    return [
        # use half precision for speed and memory savings in CI environment
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "1024",
        "--enforce-eager",
        "--max-num-seqs",
        "128",
    ]
```
**EN:** This fixture prepares `default_server_args` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `default_server_args`。

### Fixture / 夹具: server (L48-L62)
```python
@pytest.fixture(
    scope="module",
    params=[
        "",
        "--enable-chunked-prefill",
        f"--show-hidden-metrics-for-version={PREV_MINOR_VERSION}",
    ],
)
def server(model_key, default_server_args, request):
    if request.param:
        default_server_args.append(request.param)

    model_name = MODELS[model_key]
    with RemoteOpenAIServer(model_name, default_server_args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. Key inputs are `model_key`, `default_server_args`, `request`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 关键输入包括 `model_key`、`default_server_args`、`request`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L65-L68)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as cl:
        yield cl
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Module setup / 模块级配置: _PROMPT (L71-L71)
```python
_PROMPT = "Hello my name is Robert and I love magic"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `_PROMPT`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `_PROMPT`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _get_expected_values (L74-L111)
```python
def _get_expected_values(num_requests: int, prompt_ids: list[int], max_tokens: int):
    num_prompt_tokens = len(prompt_ids)

    # {metric_family: [(suffix, expected_value)]}
    return {
        "vllm:time_to_first_token_seconds": [("_count", num_requests)],
        "vllm:inter_token_latency_seconds": [
            ("_count", num_requests * (max_tokens - 1))
        ],
        "vllm:e2e_request_latency_seconds": [("_count", num_requests)],
        "vllm:request_queue_time_seconds": [("_count", num_requests)],
        "vllm:request_inference_time_seconds": [("_count", num_requests)],
        "vllm:request_prefill_time_seconds": [("_count", num_requests)],
        "vllm:request_decode_time_seconds": [("_count", num_requests)],
        "vllm:request_prompt_tokens": [
            ("_sum", num_requests * num_prompt_tokens),
            ("_count", num_requests),
        ],
# ... 12 lines omitted for brevity ...
                num_requests * (num_prompt_tokens + max_tokens),
            ),
            ("_count", num_requests * max_tokens),
        ],
        "vllm:prompt_tokens": [("_total", num_requests * num_prompt_tokens)],
        "vllm:generation_tokens": [("_total", num_requests * max_tokens)],
        "vllm:request_success": [("_total", num_requests)],
    }
```
**EN:** This helper encapsulates reusable logic in `_get_expected_values`. Key inputs are `num_requests`, `prompt_ids`, `max_tokens`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_get_expected_values` 中。 关键输入包括 `num_requests`、`prompt_ids`、`max_tokens`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_metrics_counts (L114-L179)
```python
@pytest.mark.asyncio
async def test_metrics_counts(
    server: RemoteOpenAIServer,
    client: openai.AsyncClient,
    model_key: str,
):
    if model_key == "multimodal":
        pytest.skip("Unnecessary test")

    model_name = MODELS[model_key]
    tokenizer = AutoTokenizer.from_pretrained(model_name)
    prompt_ids = tokenizer.encode(_PROMPT)
    num_requests = 10
    max_tokens = 10

    for _ in range(num_requests):
        # sending a request triggers the metrics to be logged.
        await client.completions.create(
# ... 40 lines omitted for brevity ...
                            )
                            break
                    assert found_suffix, (
                        f"Did not find {metric_name_w_suffix} in prom endpoint"
                    )
                break

        assert found_metric, f"Did not find {metric_family} in prom endpoint"
```
**EN:** This async test validates `test_metrics_counts`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `client`, `model_key`. It drives client-facing request creation through the API surface under test. The logic can skip unsupported environments when prerequisites are not satisfied. The main assertion is `response.status_code == HTTPStatus.OK` and `found_metric`.
**CN:** 这个异步测试验证 `test_metrics_counts`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`client`、`model_key`。 它通过被测 API 表面触发面向客户端的请求创建流程。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 核心断言是 `response.status_code == HTTPStatus.OK` and `found_metric`。

### Module setup / 模块级配置: EXPECTED_METRICS_V1, EXPECTED_METRICS_MM, HIDDEN_DEPRECATED_METRICS (L182-L235)
```python
EXPECTED_METRICS_V1 = [
    "vllm:num_requests_running",
    "vllm:num_requests_waiting",
    "vllm:num_requests_waiting_by_reason",
    "vllm:kv_cache_usage_perc",
    "vllm:prefix_cache_queries",
    "vllm:prefix_cache_hits",
    "vllm:num_preemptions_total",
    "vllm:prompt_tokens_total",
    "vllm:generation_tokens_total",
    "vllm:iteration_tokens_total",
    "vllm:cache_config_info",
    "vllm:request_success_total",
    "vllm:request_prompt_tokens_sum",
    "vllm:request_prompt_tokens_bucket",
    "vllm:request_prompt_tokens_count",
# ... 32 lines omitted for brevity ...
EXPECTED_METRICS_MM = [
    "vllm:mm_cache_queries",
    "vllm:mm_cache_hits",
]

HIDDEN_DEPRECATED_METRICS: list[str] = []
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `EXPECTED_METRICS_V1`, `EXPECTED_METRICS_MM`, `HIDDEN_DEPRECATED_METRICS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `EXPECTED_METRICS_V1`、`EXPECTED_METRICS_MM`、`HIDDEN_DEPRECATED_METRICS`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_metrics_exist (L238-L290)
```python
@pytest.mark.asyncio
async def test_metrics_exist(
    local_asset_server: LocalAssetServer,
    server: RemoteOpenAIServer,
    client: openai.AsyncClient,
    model_key: str,
):
    model_name = MODELS[model_key]

    # sending a request triggers the metrics to be logged.
    if model_key == "text":
        await client.completions.create(
            model=model_name,
            prompt="Hello, my name is",
            max_tokens=5,
            temperature=0.0,
        )
    else:
# ... 27 lines omitted for brevity ...
    if model_key == "multimodal":
        # NOTE: Don't use in-place assignment
        expected_metrics = expected_metrics + EXPECTED_METRICS_MM

    for metric in expected_metrics:
        if metric in HIDDEN_DEPRECATED_METRICS and not server.show_hidden_metrics:
            continue
        assert metric in response.text
```
**EN:** This async test validates `test_metrics_exist`. Relevant pytest markers include `asyncio`. Key inputs are `local_asset_server`, `server`, `client`, `model_key`. It drives client-facing request creation through the API surface under test. The main assertion is `response.status_code == HTTPStatus.OK` and `metric in response.text`.
**CN:** 这个异步测试验证 `test_metrics_exist`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `local_asset_server`、`server`、`client`、`model_key`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response.status_code == HTTPStatus.OK` and `metric in response.text`。

### Test / 测试: test_abort_metrics_reset (L293-L375)
```python
@pytest.mark.asyncio
async def test_abort_metrics_reset(
    server: RemoteOpenAIServer,
    client: openai.AsyncClient,
    model_key: str,
):
    model_name = MODELS[model_key]
    tokenizer = AutoTokenizer.from_pretrained(model_name)
    prompt_ids = tokenizer.encode(_PROMPT)

    running_requests, waiting_requests, kv_cache_usage = _get_running_metrics_from_api(
        server,
    )

    # Expect no running requests or kvcache usage
    assert running_requests == 0
    assert waiting_requests == 0
    assert kv_cache_usage == 0.0
# ... 57 lines omitted for brevity ...
        f"Expected 0 running requests after abort, got {running_requests_after}"
    )
    assert waiting_requests_after == 0, (
        f"Expected 0 waiting requests after abort, got {waiting_requests_after}"
    )
    assert kv_cache_usage_after == 0, (
        f"Expected 0% KV cache usage after abort, got {kv_cache_usage_after}"
    )
```
**EN:** This async test validates `test_abort_metrics_reset`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `client`, `model_key`. The body fans out concurrent work via `asyncio.gather`. It drives client-facing request creation through the API surface under test. The main assertion is `running_requests == 0` and `waiting_requests == 0`.
**CN:** 这个异步测试验证 `test_abort_metrics_reset`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`client`、`model_key`。 函数体通过 `asyncio.gather` 并发展开工作负载。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `running_requests == 0` and `waiting_requests == 0`。

### Helper / 辅助函数: _poll_until (L378-L387)
```python
async def _poll_until(
    predicate, *, timeout: float, interval: float = 0.5, description: str = "condition"
):
    """Poll until predicate() returns True, or raise TimeoutError."""
    start = time.time()
    while time.time() - start < timeout:
        if predicate():
            return
        await asyncio.sleep(interval)
    raise TimeoutError(f"Timed out after {timeout}s waiting for: {description}")
```
**EN:** This async helper encapsulates reusable logic in `_poll_until`. Key inputs are `predicate`.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `_poll_until` 中。 关键输入包括 `predicate`。

### Helper / 辅助函数: _get_running_metrics_from_api (L390-L422)
```python
def _get_running_metrics_from_api(server: RemoteOpenAIServer):
    """Return (running_count, waiting_count, kv_cache_usage)"""

    response = requests.get(server.url_for("metrics"))
    assert response.status_code == HTTPStatus.OK

    # Verify running and waiting requests counts and KV cache usage are zero
    running_requests, waiting_requests, kv_cache_usage = None, None, None

    kv_cache_usage_metric = "vllm:kv_cache_usage_perc"

    for family in text_string_to_metric_families(response.text):
        if family.name == "vllm:num_requests_running":
            for sample in family.samples:
                if sample.name == "vllm:num_requests_running":
                    running_requests = sample.value
                    break
        elif family.name == "vllm:num_requests_waiting":
# ... 7 lines omitted for brevity ...
                    kv_cache_usage = sample.value
                    break

    assert running_requests is not None
    assert waiting_requests is not None
    assert kv_cache_usage is not None

    return running_requests, waiting_requests, kv_cache_usage
```
**EN:** This helper encapsulates reusable logic in `_get_running_metrics_from_api`. Key inputs are `server`. It returns computed state or helper objects back to the caller. The main assertion is `response.status_code == HTTPStatus.OK` and `running_requests is not None`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_get_running_metrics_from_api` 中。 关键输入包括 `server`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `response.status_code == HTTPStatus.OK` and `running_requests is not None`。

### Test / 测试: test_metrics_exist_run_batch (L425-L485)
```python
def test_metrics_exist_run_batch():
    input_batch = """{"custom_id": "request-0", "method": "POST", "url": "/v1/embeddings", "body": {"model": "intfloat/multilingual-e5-small", "input": "You are a helpful assistant."}}"""  # noqa: E501

    base_url = "0.0.0.0"
    port = str(get_open_port())
    server_url = f"http://{base_url}:{port}"

    with (
        tempfile.NamedTemporaryFile("w") as input_file,
        tempfile.NamedTemporaryFile("r") as output_file,
    ):
        input_file.write(input_batch)
        input_file.flush()
        proc = subprocess.Popen(
            [
                sys.executable,
                "-m",
                "vllm.entrypoints.openai.run_batch",
# ... 35 lines omitted for brevity ...
            assert response.status_code == HTTPStatus.OK
        finally:
            proc.terminate()
            try:
                proc.wait(timeout=15)
            except subprocess.TimeoutExpired:
                proc.kill()
                proc.wait(timeout=5)
```
**EN:** This test validates `test_metrics_exist_run_batch`. The main assertion is `response.status_code == HTTPStatus.OK`.
**CN:** 这个测试验证 `test_metrics_exist_run_batch`。 核心断言是 `response.status_code == HTTPStatus.OK`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `asyncio`, `http.HTTPStatus`, `subprocess`, `sys`, `tempfile`, `time`
- **Third-party / 第三方**: `openai`, `prometheus_client.parser.text_string_to_metric_families`, `pytest`, `pytest_asyncio`, `requests`, `transformers.AutoTokenizer`
- **Project / 项目内**: `tests.conftest.LocalAssetServer`, `tests.utils.RemoteOpenAIServer`, `vllm.utils.network_utils.get_open_port`, `vllm.version`
