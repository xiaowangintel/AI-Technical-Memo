# test_bi_encoder_online.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/scoring/test_bi_encoder_online.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and scoring or reranking behavior. The file defines 14 test(s), 2 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与打分或重排行为。它定义了 14 个测试、2 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L11)
```python
import pytest
import requests

from tests.entrypoints.pooling.scoring.util import EncoderScoringHfRunner
from tests.utils import RemoteOpenAIServer
from vllm.entrypoints.pooling.pooling.protocol import PoolingResponse
from vllm.entrypoints.pooling.scoring.protocol import RerankResponse, ScoreResponse
from vllm.platforms import current_platform
```
**EN:** Imports third-party packages like `pytest`, `requests`, project helpers such as `tests.entrypoints.pooling.scoring.util.EncoderScoringHfRunner`, `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.pooling.pooling.protocol.PoolingResponse`.
**CN:** 导入第三方包（如 `pytest`、`requests`）、项目内辅助模块（如 `tests.entrypoints.pooling.scoring.util.EncoderScoringHfRunner`、`tests.utils.RemoteOpenAIServer`、`vllm.entrypoints.pooling.pooling.protocol.PoolingResponse`）。

### Module setup / 模块级配置: MODEL_NAME, input_text, DTYPE (L13-L26)
```python
MODEL_NAME = "BAAI/bge-base-en-v1.5"
input_text = "This product was excellent and exceeded my expectations"
DTYPE = "half"
EMBEDDING_SIZE = 768

TEXTS_1 = [
    "What is the capital of France?",
    "What is the capital of Germany?",
]

TEXTS_2 = [
    "The capital of France is Paris.",
    "The capital of Germany is Berlin.",
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `input_text`, `DTYPE`, `EMBEDDING_SIZE`, `TEXTS_1`, `TEXTS_2`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`input_text`、`DTYPE`、`EMBEDDING_SIZE`、`TEXTS_1`、`TEXTS_2`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L29-L38)
```python
@pytest.fixture(scope="module")
def server():
    args = ["--enforce-eager", "--max-model-len", "100", "--dtype", DTYPE]

    # ROCm: Use Flex Attention to support encoder-only self-attention.
    if current_platform.is_rocm():
        args.extend(["--attention-backend", "FLEX_ATTENTION"])

    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: hf_model (L41-L43)
```python
@pytest.fixture(scope="module")
def hf_model():
    return EncoderScoringHfRunner(MODEL_NAME)
```
**EN:** This fixture prepares `hf_model` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `hf_model`。

### Test / 测试: test_score_api_queries_str_1_documents_str_1 (L46-L69)
```python
@pytest.mark.asyncio
async def test_score_api_queries_str_1_documents_str_1(
    hf_model, server: RemoteOpenAIServer
):
    score_response = requests.post(
        server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "queries": TEXTS_1[0],
            "documents": TEXTS_2[0],
        },
    )
    score_response.raise_for_status()
    score = ScoreResponse.model_validate(score_response.json())

    assert score.id is not None
    assert score.data is not None
    assert len(score.data) == 1

    vllm_outputs = [d.score for d in score.data]
    hf_outputs = hf_model.predict([[TEXTS_1[0], TEXTS_2[0]]]).tolist()

    for i in range(len(vllm_outputs)):
        assert hf_outputs[i] == pytest.approx(vllm_outputs[i], rel=0.01)
```
**EN:** This async test validates `test_score_api_queries_str_1_documents_str_1`. Relevant pytest markers include `asyncio`. Key inputs are `hf_model`, `server`. The main assertion is `score.id is not None` and `score.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_queries_str_1_documents_str_1`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `hf_model`、`server`。 核心断言是 `score.id is not None` and `score.data is not None`。

### Test / 测试: test_score_api_queries_str_1_documents_str_n (L72-L100)
```python
@pytest.mark.asyncio
async def test_score_api_queries_str_1_documents_str_n(
    hf_model, server: RemoteOpenAIServer
):
    text_pairs = [
        [TEXTS_1[0], TEXTS_2[0]],
        [TEXTS_1[0], TEXTS_2[1]],
    ]

    score_response = requests.post(
        server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "queries": TEXTS_1[0],
            "documents": TEXTS_2,
        },
    )
    score_response.raise_for_status()
    score = ScoreResponse.model_validate(score_response.json())

    assert score.id is not None
    assert score.data is not None
    assert len(score.data) == 2

    vllm_outputs = [d.score for d in score.data]
    hf_outputs = hf_model.predict(text_pairs).tolist()

    for i in range(len(vllm_outputs)):
        assert hf_outputs[i] == pytest.approx(vllm_outputs[i], rel=0.01)
```
**EN:** This async test validates `test_score_api_queries_str_1_documents_str_n`. Relevant pytest markers include `asyncio`. Key inputs are `hf_model`, `server`. The main assertion is `score.id is not None` and `score.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_queries_str_1_documents_str_n`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `hf_model`、`server`。 核心断言是 `score.id is not None` and `score.data is not None`。

### Test / 测试: test_score_api_queries_str_n_documents_str_n (L103-L131)
```python
@pytest.mark.asyncio
async def test_score_api_queries_str_n_documents_str_n(
    hf_model, server: RemoteOpenAIServer
):
    text_pairs = [
        [TEXTS_1[0], TEXTS_2[0]],
        [TEXTS_1[1], TEXTS_2[1]],
    ]

    score_response = requests.post(
        server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "queries": TEXTS_1,
            "documents": TEXTS_2,
        },
    )
    score_response.raise_for_status()
    score = ScoreResponse.model_validate(score_response.json())

    assert score.id is not None
    assert score.data is not None
    assert len(score.data) == 2

    vllm_outputs = [d.score for d in score.data]
    hf_outputs = hf_model.predict(text_pairs).tolist()

    for i in range(len(vllm_outputs)):
        assert hf_outputs[i] == pytest.approx(vllm_outputs[i], rel=0.01)
```
**EN:** This async test validates `test_score_api_queries_str_n_documents_str_n`. Relevant pytest markers include `asyncio`. Key inputs are `hf_model`, `server`. The main assertion is `score.id is not None` and `score.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_queries_str_n_documents_str_n`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `hf_model`、`server`。 核心断言是 `score.id is not None` and `score.data is not None`。

### Test / 测试: test_score_api_queries_vs_documents (L134-L160)
```python
@pytest.mark.asyncio
async def test_score_api_queries_vs_documents(hf_model, server: RemoteOpenAIServer):
    text_pairs = [
        [TEXTS_1[0], TEXTS_2[0]],
        [TEXTS_1[1], TEXTS_2[1]],
    ]

    score_response = requests.post(
        server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "queries": TEXTS_1,
            "documents": TEXTS_2,
        },
    )
    score_response.raise_for_status()
    score = ScoreResponse.model_validate(score_response.json())

    assert score.id is not None
    assert score.data is not None
    assert len(score.data) == 2

    vllm_outputs = [d.score for d in score.data]
    hf_outputs = hf_model.predict(text_pairs).tolist()

    for i in range(len(vllm_outputs)):
        assert hf_outputs[i] == pytest.approx(vllm_outputs[i], rel=0.01)
```
**EN:** This async test validates `test_score_api_queries_vs_documents`. Relevant pytest markers include `asyncio`. Key inputs are `hf_model`, `server`. The main assertion is `score.id is not None` and `score.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_queries_vs_documents`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `hf_model`、`server`。 核心断言是 `score.id is not None` and `score.data is not None`。

### Test / 测试: test_score_api_queries_vs_items (L163-L189)
```python
@pytest.mark.asyncio
async def test_score_api_queries_vs_items(hf_model, server: RemoteOpenAIServer):
    text_pairs = [
        [TEXTS_1[0], TEXTS_2[0]],
        [TEXTS_1[1], TEXTS_2[1]],
    ]

    score_response = requests.post(
        server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "queries": TEXTS_1,
            "items": TEXTS_2,
        },
    )
    score_response.raise_for_status()
    score = ScoreResponse.model_validate(score_response.json())

    assert score.id is not None
    assert score.data is not None
    assert len(score.data) == 2

    vllm_outputs = [d.score for d in score.data]
    hf_outputs = hf_model.predict(text_pairs).tolist()

    for i in range(len(vllm_outputs)):
        assert hf_outputs[i] == pytest.approx(vllm_outputs[i], rel=0.01)
```
**EN:** This async test validates `test_score_api_queries_vs_items`. Relevant pytest markers include `asyncio`. Key inputs are `hf_model`, `server`. The main assertion is `score.id is not None` and `score.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_queries_vs_items`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `hf_model`、`server`。 核心断言是 `score.id is not None` and `score.data is not None`。

### Test / 测试: test_score_api_text_1_vs_text_2 (L192-L218)
```python
@pytest.mark.asyncio
async def test_score_api_text_1_vs_text_2(hf_model, server: RemoteOpenAIServer):
    text_pairs = [
        [TEXTS_1[0], TEXTS_2[0]],
        [TEXTS_1[1], TEXTS_2[1]],
    ]

    score_response = requests.post(
        server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "text_1": TEXTS_1,
            "text_2": TEXTS_2,
        },
    )
    score_response.raise_for_status()
    score = ScoreResponse.model_validate(score_response.json())

    assert score.id is not None
    assert score.data is not None
    assert len(score.data) == 2

    vllm_outputs = [d.score for d in score.data]
    hf_outputs = hf_model.predict(text_pairs).tolist()

    for i in range(len(vllm_outputs)):
        assert hf_outputs[i] == pytest.approx(vllm_outputs[i], rel=0.01)
```
**EN:** This async test validates `test_score_api_text_1_vs_text_2`. Relevant pytest markers include `asyncio`. Key inputs are `hf_model`, `server`. The main assertion is `score.id is not None` and `score.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_text_1_vs_text_2`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `hf_model`、`server`。 核心断言是 `score.id is not None` and `score.data is not None`。

### Test / 测试: test_score_api_data_1_vs_data_2 (L221-L247)
```python
@pytest.mark.asyncio
async def test_score_api_data_1_vs_data_2(hf_model, server: RemoteOpenAIServer):
    text_pairs = [
        [TEXTS_1[0], TEXTS_2[0]],
        [TEXTS_1[1], TEXTS_2[1]],
    ]

    score_response = requests.post(
        server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "data_1": TEXTS_1,
            "data_2": TEXTS_2,
        },
    )
    score_response.raise_for_status()
    score = ScoreResponse.model_validate(score_response.json())

    assert score.id is not None
    assert score.data is not None
    assert len(score.data) == 2

    vllm_outputs = [d.score for d in score.data]
    hf_outputs = hf_model.predict(text_pairs).tolist()

    for i in range(len(vllm_outputs)):
        assert hf_outputs[i] == pytest.approx(vllm_outputs[i], rel=0.01)
```
**EN:** This async test validates `test_score_api_data_1_vs_data_2`. Relevant pytest markers include `asyncio`. Key inputs are `hf_model`, `server`. The main assertion is `score.id is not None` and `score.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_data_1_vs_data_2`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `hf_model`、`server`。 核心断言是 `score.id is not None` and `score.data is not None`。

### Test / 测试: test_rerank_api_texts (L250-L274)
```python
@pytest.mark.asyncio
async def test_rerank_api_texts(server: RemoteOpenAIServer):
    query = "What is the capital of France?"
    documents = [
        "The capital of Brazil is Brasilia.",
        "The capital of France is Paris.",
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
    paris_result = next(r for r in rerank.results if r.index == 1)
    brazil_result = next(r for r in rerank.results if r.index == 0)
    assert paris_result.relevance_score > brazil_result.relevance_score
```
**EN:** This async test validates `test_rerank_api_texts`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `rerank.id is not None` and `rerank.results is not None`.
**CN:** 这个异步测试验证 `test_rerank_api_texts`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `rerank.id is not None` and `rerank.results is not None`。

### Test / 测试: test_rerank_api_top_n (L277-L296)
```python
@pytest.mark.asyncio
async def test_rerank_api_top_n(server: RemoteOpenAIServer):
    query = "What is the capital of France?"
    documents = [
        "The capital of Brazil is Brasilia.",
        "The capital of France is Paris.",
        "Cross-encoder models are neat",
    ]

    rerank_response = requests.post(
        server.url_for("rerank"),
        json={"model": MODEL_NAME, "query": query, "documents": documents, "top_n": 2},
    )
    rerank_response.raise_for_status()
    rerank = RerankResponse.model_validate(rerank_response.json())

    assert rerank.id is not None
    assert rerank.results is not None
    assert len(rerank.results) == 2
    assert rerank.results[0].index == 1
```
**EN:** This async test validates `test_rerank_api_top_n`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `rerank.id is not None` and `rerank.results is not None`.
**CN:** 这个异步测试验证 `test_rerank_api_top_n`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `rerank.id is not None` and `rerank.results is not None`。

### Test / 测试: test_rerank_api_max_model_len (L299-L313)
```python
@pytest.mark.asyncio
async def test_rerank_api_max_model_len(server: RemoteOpenAIServer):
    query = "What is the capital of France?" * 100
    documents = [
        "The capital of Brazil is Brasilia.",
        "The capital of France is Paris.",
    ]

    rerank_response = requests.post(
        server.url_for("rerank"),
        json={"model": MODEL_NAME, "query": query, "documents": documents},
    )
    assert rerank_response.status_code == 400
    # Assert just a small fragments of the response
    assert "Please reduce the length of the input prompt" in rerank_response.text
```
**EN:** This async test validates `test_rerank_api_max_model_len`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `rerank_response.status_code == 400` and `'Please reduce the length of the input prompt' in rerank_response.text`.
**CN:** 这个异步测试验证 `test_rerank_api_max_model_len`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `rerank_response.status_code == 400` and `'Please reduce the length of the input prompt' in rerank_response.text`。

### Test / 测试: test_score_api_max_model_len (L316-L347)
```python
@pytest.mark.asyncio
async def test_score_api_max_model_len(server: RemoteOpenAIServer):
    queries = "What is the capital of France?" * 20
    documents = [
        "The capital of Brazil is Brasilia.",
        "The capital of France is Paris.",
    ]

    score_response = requests.post(
        server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "queries": queries,
            "documents": documents,
        },
    )
    assert score_response.status_code == 400
    # Assert just a small fragments of the response
# ... 6 lines omitted for brevity ...
            "model": MODEL_NAME,
            "queries": queries,
            "documents": documents,
            "truncate_prompt_tokens": 101,
        },
    )
    assert score_response.status_code == 400
    assert "Please request a smaller truncation size." in score_response.text
```
**EN:** This async test validates `test_score_api_max_model_len`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `score_response.status_code == 400` and `'Please reduce the length of the input prompt' in score_response.text`.
**CN:** 这个异步测试验证 `test_score_api_max_model_len`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `score_response.status_code == 400` and `'Please reduce the length of the input prompt' in score_response.text`。

### Test / 测试: test_invocations (L350-L382)
```python
@pytest.mark.asyncio
async def test_invocations(server: RemoteOpenAIServer):
    query = "What is the capital of France?"
    documents = [
        "The capital of Brazil is Brasilia.",
        "The capital of France is Paris.",
    ]

    request_args = {
        "model": MODEL_NAME,
        "query": query,
        "documents": documents,
    }

    rerank_response = requests.post(server.url_for("rerank"), json=request_args)
    rerank_response.raise_for_status()

    invocation_response = requests.post(
# ... 7 lines omitted for brevity ...
    assert rerank_output.keys() == invocation_output.keys()
    for rerank_result, invocations_result in zip(
        rerank_output["results"], invocation_output["results"]
    ):
        assert rerank_result.keys() == invocations_result.keys()
        assert rerank_result["relevance_score"] == pytest.approx(
            invocations_result["relevance_score"], rel=0.01
        )
```
**EN:** This async test validates `test_invocations`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `rerank_output.keys() == invocation_output.keys()` and `rerank_result.keys() == invocations_result.keys()`.
**CN:** 这个异步测试验证 `test_invocations`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `rerank_output.keys() == invocation_output.keys()` and `rerank_result.keys() == invocations_result.keys()`。

### Test / 测试: test_pooling_embed (L385-L398)
```python
@pytest.mark.asyncio
async def test_pooling_embed(server: RemoteOpenAIServer):
    response = requests.post(
        server.url_for("pooling"),
        json={
            "model": MODEL_NAME,
            "input": input_text,
            "encoding_format": "float",
            "task": "embed",
        },
    )
    poolings = PoolingResponse.model_validate(response.json())
    assert len(poolings.data) == 1
    assert len(poolings.data[0].data) == EMBEDDING_SIZE
```
**EN:** This async test validates `test_pooling_embed`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `len(poolings.data) == 1` and `len(poolings.data[0].data) == EMBEDDING_SIZE`.
**CN:** 这个异步测试验证 `test_pooling_embed`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `len(poolings.data) == 1` and `len(poolings.data[0].data) == EMBEDDING_SIZE`。

### Test / 测试: test_pooling_not_supported (L401-L418)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("task", ["classify", "token_classify", "plugin"])
async def test_pooling_not_supported(server: RemoteOpenAIServer, task: str):
    response = requests.post(
        server.url_for("pooling"),
        json={
            "model": MODEL_NAME,
            "input": input_text,
            "encoding_format": "float",
            "task": task,
        },
    )
    assert response.json()["error"]["type"] == "BadRequestError"
    if task == "plugin":
        err_msg = "No IOProcessor plugin installed."
    else:
        err_msg = f"Unsupported task: {task!r}"
    assert response.json()["error"]["message"].startswith(err_msg)
```
**EN:** This async test validates `test_pooling_not_supported`. It uses parameterization over `task`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `task`. The main assertion is `response.json()['error']['type'] == 'BadRequestError'` and `response.json()['error']['message'].startswith(err_msg)`.
**CN:** 这个异步测试验证 `test_pooling_not_supported`。 它通过参数化组合 `task`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`task`。 核心断言是 `response.json()['error']['type'] == 'BadRequestError'` and `response.json()['error']['message'].startswith(err_msg)`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `pytest`, `requests`
- **Project / 项目内**: `tests.entrypoints.pooling.scoring.util.EncoderScoringHfRunner`, `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.pooling.pooling.protocol.PoolingResponse`, `vllm.entrypoints.pooling.scoring.protocol.RerankResponse`, `vllm.entrypoints.pooling.scoring.protocol.ScoreResponse`, `vllm.platforms.current_platform`
