# test_late_interaction_online_vision.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/scoring/test_late_interaction_online_vision.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and scoring or reranking behavior. The file defines 6 test(s), 1 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与打分或重排行为。它定义了 6 个测试、1 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L12)
```python
import pytest
import requests

from tests.entrypoints.pooling.scoring.util import (
    make_base64_image,
    make_image_mm_param,
)
from tests.utils import RemoteOpenAIServer
from vllm.entrypoints.pooling.scoring.protocol import RerankResponse, ScoreResponse
```
**EN:** Imports third-party packages like `pytest`, `requests`, project helpers such as `tests.entrypoints.pooling.scoring.util.make_base64_image`, `tests.entrypoints.pooling.scoring.util.make_image_mm_param`, `tests.utils.RemoteOpenAIServer`.
**CN:** 导入第三方包（如 `pytest`、`requests`）、项目内辅助模块（如 `tests.entrypoints.pooling.scoring.util.make_base64_image`、`tests.entrypoints.pooling.scoring.util.make_image_mm_param`、`tests.utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: MODEL_NAME (L14-L14)
```python
MODEL_NAME = "vidore/colpali-v1.3-hf"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L17-L20)
```python
@pytest.fixture(scope="module")
def server():
    with RemoteOpenAIServer(MODEL_NAME, []) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_score_api_query_text_vs_docs_image (L23-L49)
```python
@pytest.mark.asyncio
async def test_score_api_query_text_vs_docs_image(server: RemoteOpenAIServer):
    query = "Describe the red object"

    red_image = make_base64_image(64, 64, color=(255, 0, 0))
    blue_image = make_base64_image(64, 64, color=(0, 0, 255))

    documents = [
        make_image_mm_param(red_image),
        make_image_mm_param(blue_image),
    ]

    score_response = requests.post(
        server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "queries": query,
            "documents": documents,
        },
    )
    score_response.raise_for_status()
    scores = ScoreResponse.model_validate(score_response.json())

    assert scores.id is not None
    assert scores.data is not None
    assert len(scores.data) == 2
    assert scores.data[0].score > scores.data[1].score
```
**EN:** This async test validates `test_score_api_query_text_vs_docs_image`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `scores.id is not None` and `scores.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_query_text_vs_docs_image`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `scores.id is not None` and `scores.data is not None`。

### Test / 测试: test_score_api_query_text_vs_docs_mix (L52-L75)
```python
@pytest.mark.asyncio
async def test_score_api_query_text_vs_docs_mix(server: RemoteOpenAIServer):
    red_image = make_base64_image(64, 64, color=(255, 0, 0))
    query = "What is the capital of France?"
    documents: list = [
        "The capital of France is Paris.",
        make_image_mm_param(red_image),
    ]

    score_response = requests.post(
        server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "queries": query,
            "documents": documents,
        },
    )
    score_response.raise_for_status()
    scores = ScoreResponse.model_validate(score_response.json())

    assert scores.id is not None
    assert scores.data is not None
    assert len(scores.data) == 2
    assert scores.data[0].score > scores.data[1].score
```
**EN:** This async test validates `test_score_api_query_text_vs_docs_mix`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `scores.id is not None` and `scores.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_query_text_vs_docs_mix`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `scores.id is not None` and `scores.data is not None`。

### Test / 测试: test_score_api_query_image_vs_docs_text (L78-L102)
```python
@pytest.mark.asyncio
async def test_score_api_query_image_vs_docs_text(server: RemoteOpenAIServer):
    red_image = make_base64_image(64, 64, color=(255, 0, 0))
    image_query = make_image_mm_param(red_image, text="red color")

    documents = [
        "Describe the red object.",
        "The capital of France is Paris.",
    ]

    score_response = requests.post(
        server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "queries": image_query,
            "documents": documents,
        },
    )
    score_response.raise_for_status()
    scores = ScoreResponse.model_validate(score_response.json())

    assert scores.id is not None
    assert scores.data is not None
    assert len(scores.data) == 2
    assert scores.data[0].score > scores.data[1].score
```
**EN:** This async test validates `test_score_api_query_image_vs_docs_text`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `scores.id is not None` and `scores.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_query_image_vs_docs_text`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `scores.id is not None` and `scores.data is not None`。

### Test / 测试: test_rerank_api_query_text_vs_docs_image (L105-L132)
```python
@pytest.mark.asyncio
async def test_rerank_api_query_text_vs_docs_image(server: RemoteOpenAIServer):
    query = "Describe the red object"

    red_image = make_base64_image(64, 64, color=(255, 0, 0))
    blue_image = make_base64_image(64, 64, color=(0, 0, 255))

    documents = [
        make_image_mm_param(red_image),
        make_image_mm_param(blue_image),
    ]

    rerank_response = requests.post(
        server.url_for("rerank"),
        json={"model": MODEL_NAME, "query": query, "documents": documents},
    )

    rerank_response.raise_for_status()
    rerank = RerankResponse.model_validate(rerank_response.json())

    assert rerank.id is not None
    assert rerank.results is not None
    assert len(rerank.results) == 2

    red_result = next(r for r in rerank.results if r.index == 0)
    blue_result = next(r for r in rerank.results if r.index == 1)

    assert red_result.relevance_score > blue_result.relevance_score
```
**EN:** This async test validates `test_rerank_api_query_text_vs_docs_image`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `rerank.id is not None` and `rerank.results is not None`.
**CN:** 这个异步测试验证 `test_rerank_api_query_text_vs_docs_image`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `rerank.id is not None` and `rerank.results is not None`。

### Test / 测试: test_rerank_api_query_text_vs_docs_mix (L135-L162)
```python
@pytest.mark.asyncio
async def test_rerank_api_query_text_vs_docs_mix(server: RemoteOpenAIServer):
    red_image = make_base64_image(64, 64, color=(255, 0, 0))
    query = "What is the capital of France?"
    documents: list = [
        "The capital of France is Paris.",
        make_image_mm_param(red_image),
    ]

    rerank_response = requests.post(
        server.url_for("rerank"),
        json={
            "model": MODEL_NAME,
            "query": query,
            "documents": documents,
        },
    )
    rerank_response.raise_for_status()
    rerank = RerankResponse.model_validate(rerank_response.json())

    assert rerank.id is not None
    assert rerank.results is not None
    assert len(rerank.results) == 2

    result0 = next(r for r in rerank.results if r.index == 0)
    result1 = next(r for r in rerank.results if r.index == 1)

    assert result0.relevance_score > result1.relevance_score
```
**EN:** This async test validates `test_rerank_api_query_text_vs_docs_mix`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `rerank.id is not None` and `rerank.results is not None`.
**CN:** 这个异步测试验证 `test_rerank_api_query_text_vs_docs_mix`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `rerank.id is not None` and `rerank.results is not None`。

### Test / 测试: test_rerank_api_query_image_vs_docs_text (L165-L193)
```python
@pytest.mark.asyncio
async def test_rerank_api_query_image_vs_docs_text(server: RemoteOpenAIServer):
    red_image = make_base64_image(64, 64, color=(255, 0, 0))
    image_query = make_image_mm_param(red_image, text="red color")

    documents = [
        "Describe the red object.",
        "The capital of France is Paris.",
    ]

    rerank_response = requests.post(
        server.url_for("rerank"),
        json={
            "model": MODEL_NAME,
            "query": image_query,
            "documents": documents,
        },
    )
    rerank_response.raise_for_status()
    rerank = RerankResponse.model_validate(rerank_response.json())

    assert rerank.id is not None
    assert rerank.results is not None
    assert len(rerank.results) == 2

    result0 = next(r for r in rerank.results if r.index == 0)
    result1 = next(r for r in rerank.results if r.index == 1)

    assert result0.relevance_score > result1.relevance_score
```
**EN:** This async test validates `test_rerank_api_query_image_vs_docs_text`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `rerank.id is not None` and `rerank.results is not None`.
**CN:** 这个异步测试验证 `test_rerank_api_query_image_vs_docs_text`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `rerank.id is not None` and `rerank.results is not None`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `pytest`, `requests`
- **Project / 项目内**: `tests.entrypoints.pooling.scoring.util.make_base64_image`, `tests.entrypoints.pooling.scoring.util.make_image_mm_param`, `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.pooling.scoring.protocol.RerankResponse`, `vllm.entrypoints.pooling.scoring.protocol.ScoreResponse`
