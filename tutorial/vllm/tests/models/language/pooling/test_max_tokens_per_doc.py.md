# test_max_tokens_per_doc.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/pooling/test_max_tokens_per_doc.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and language-model behavior. The file defines 4 test(s), 1 fixture(s), and 2 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与语言模型行为。它定义了 4 个测试、1 个 fixture，以及 2 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L7-L15)
```python
import json
import os
from dataclasses import dataclass

import pytest
import requests

from tests.utils import VLLM_PATH, RemoteOpenAIServer
from vllm.entrypoints.pooling.scoring.protocol import RerankResponse
```
**EN:** Imports standard-library modules such as `dataclasses.dataclass`, `json`, `os`, third-party packages like `pytest`, `requests`, project helpers such as `tests.utils.RemoteOpenAIServer`, `tests.utils.VLLM_PATH`, `vllm.entrypoints.pooling.scoring.protocol.RerankResponse`.
**CN:** 导入标准库模块（如 `dataclasses.dataclass`、`json`、`os`）、第三方包（如 `pytest`、`requests`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`tests.utils.VLLM_PATH`、`vllm.entrypoints.pooling.scoring.protocol.RerankResponse`）。

### Module setup / 模块级配置: TEMPLATE_DIR, ExpectedPromptTokens, long_query (L17-L23)
```python
os.environ["VLLM_LOGGING_LEVEL"] = "WARNING"

TEMPLATE_DIR = str(VLLM_PATH / "examples/pooling/score/template")
ExpectedPromptTokens = int | tuple[int, ...]

long_query = "What is the capital of France?" * 20
long_doc = "The capital of France is Paris. " * 20
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `TEMPLATE_DIR`, `ExpectedPromptTokens`, `long_query`, `long_doc`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `TEMPLATE_DIR`、`ExpectedPromptTokens`、`long_query`、`long_doc`。后续辅助函数和测试会复用这些值，以减少重复。

### Class / 类: TestConfig (L26-L33)
```python
@dataclass
class TestConfig:
    model: str
    args: list[str]
    without_truncated_prompt_tokens: ExpectedPromptTokens
    with_max_tokens_per_query_prompt_tokens: ExpectedPromptTokens
    with_max_tokens_per_doc_prompt_tokens: ExpectedPromptTokens
    with_max_tokens_per_query_and_doc_prompt_tokens: ExpectedPromptTokens
```
**EN:** This class groups related scenarios in `TestConfig`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `TestConfig` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Module setup / 模块级配置: RERANK_CONFIGS (L36-L118)
```python
RERANK_CONFIGS = [
    # 1. cross-encoder
    TestConfig(
        model="jinaai/jina-reranker-v2-base-multilingual",
        args=[
            "--enforce-eager",
            "--max-model-len",
            "1024",
            "--trust-remote-code",
        ],
        without_truncated_prompt_tokens=284,
        with_max_tokens_per_query_prompt_tokens=154,
        with_max_tokens_per_doc_prompt_tokens=154,
        with_max_tokens_per_query_and_doc_prompt_tokens=24,
    ),
    # 2. cross-encoder + score template
# ... 61 lines omitted for brevity ...
        without_truncated_prompt_tokens=567,
        with_max_tokens_per_query_prompt_tokens=308,
        with_max_tokens_per_doc_prompt_tokens=436,
        with_max_tokens_per_query_and_doc_prompt_tokens=177,
    ),
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `RERANK_CONFIGS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `RERANK_CONFIGS`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: assert_prompt_tokens (L121-L125)
```python
def assert_prompt_tokens(actual: int, expected: ExpectedPromptTokens) -> None:
    if isinstance(expected, int):
        assert actual == expected
    else:
        assert actual in expected
```
**EN:** This helper encapsulates reusable logic in `assert_prompt_tokens`. Key inputs are `actual`, `expected`. The main assertion is `actual == expected` and `actual in expected`.
**CN:** 这个辅助函数将可复用逻辑封装在 `assert_prompt_tokens` 中。 关键输入包括 `actual`、`expected`。 核心断言是 `actual == expected` and `actual in expected`。

### Fixture / 夹具: server (L128-L132)
```python
@pytest.fixture(scope="module", params=RERANK_CONFIGS, ids=lambda c: c.model)
def server(request):
    config: TestConfig = request.param
    with RemoteOpenAIServer(config.model, config.args) as remote_server:
        yield config, remote_server
```
**EN:** This fixture prepares `server` for dependent tests. Key inputs are `request`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 关键输入包括 `request`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_without_truncated (L135-L152)
```python
def test_without_truncated(server):
    """Test that max_tokens_per_doc truncates documents correctly."""
    config, remote_server = server

    response = requests.post(
        remote_server.url_for("rerank"),
        json={"model": config.model, "query": long_query, "documents": [long_doc]},
    )
    response.raise_for_status()
    rerank = RerankResponse.model_validate(response.json())

    assert rerank.id is not None
    assert rerank.results is not None
    assert len(rerank.results) == 1
    assert_prompt_tokens(
        rerank.usage.prompt_tokens,
        config.without_truncated_prompt_tokens,
    )
```
**EN:** This test validates `test_without_truncated`. Key inputs are `server`. The main assertion is `rerank.id is not None` and `rerank.results is not None`.
**CN:** 这个测试验证 `test_without_truncated`。 关键输入包括 `server`。 核心断言是 `rerank.id is not None` and `rerank.results is not None`。

### Test / 测试: test_max_tokens_per_query (L155-L177)
```python
def test_max_tokens_per_query(server):
    """Test that max_tokens_per_doc truncates documents correctly."""
    config, remote_server = server

    response = requests.post(
        remote_server.url_for("rerank"),
        json={
            "model": config.model,
            "query": long_query,
            "documents": [long_doc],
            "max_tokens_per_query": 10,
        },
    )
    response.raise_for_status()
    rerank = RerankResponse.model_validate(response.json())

    assert rerank.id is not None
    assert rerank.results is not None
    assert len(rerank.results) == 1
    assert_prompt_tokens(
        rerank.usage.prompt_tokens,
        config.with_max_tokens_per_query_prompt_tokens,
    )
```
**EN:** This test validates `test_max_tokens_per_query`. Key inputs are `server`. The main assertion is `rerank.id is not None` and `rerank.results is not None`.
**CN:** 这个测试验证 `test_max_tokens_per_query`。 关键输入包括 `server`。 核心断言是 `rerank.id is not None` and `rerank.results is not None`。

### Test / 测试: test_max_tokens_per_doc (L180-L202)
```python
def test_max_tokens_per_doc(server):
    """Test that max_tokens_per_doc truncates documents correctly."""
    config, remote_server = server

    response = requests.post(
        remote_server.url_for("rerank"),
        json={
            "model": config.model,
            "query": long_query,
            "documents": [long_doc],
            "max_tokens_per_doc": 10,
        },
    )
    response.raise_for_status()
    rerank = RerankResponse.model_validate(response.json())

    assert rerank.id is not None
    assert rerank.results is not None
    assert len(rerank.results) == 1
    assert_prompt_tokens(
        rerank.usage.prompt_tokens,
        config.with_max_tokens_per_doc_prompt_tokens,
    )
```
**EN:** This test validates `test_max_tokens_per_doc`. Key inputs are `server`. The main assertion is `rerank.id is not None` and `rerank.results is not None`.
**CN:** 这个测试验证 `test_max_tokens_per_doc`。 关键输入包括 `server`。 核心断言是 `rerank.id is not None` and `rerank.results is not None`。

### Test / 测试: test_max_tokens_per_query_and_doc (L205-L228)
```python
def test_max_tokens_per_query_and_doc(server):
    """Test that max_tokens_per_doc truncates documents correctly."""
    config, remote_server = server

    response = requests.post(
        remote_server.url_for("rerank"),
        json={
            "model": config.model,
            "query": long_query,
            "documents": [long_doc],
            "max_tokens_per_query": 10,
            "max_tokens_per_doc": 10,
        },
    )
    response.raise_for_status()
    rerank = RerankResponse.model_validate(response.json())

    assert rerank.id is not None
    assert rerank.results is not None
    assert len(rerank.results) == 1
    assert_prompt_tokens(
        rerank.usage.prompt_tokens,
        config.with_max_tokens_per_query_and_doc_prompt_tokens,
    )
```
**EN:** This test validates `test_max_tokens_per_query_and_doc`. Key inputs are `server`. The main assertion is `rerank.id is not None` and `rerank.results is not None`.
**CN:** 这个测试验证 `test_max_tokens_per_query_and_doc`。 关键输入包括 `server`。 核心断言是 `rerank.id is not None` and `rerank.results is not None`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `dataclasses.dataclass`, `json`, `os`
- **Third-party / 第三方**: `pytest`, `requests`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `tests.utils.VLLM_PATH`, `vllm.entrypoints.pooling.scoring.protocol.RerankResponse`
