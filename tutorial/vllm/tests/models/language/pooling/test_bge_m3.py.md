# test_bge_m3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/pooling/test_bge_m3.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and language-model behavior. The file defines 4 test(s), 2 fixture(s), and 4 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与语言模型行为。它定义了 4 个测试、2 个 fixture，以及 4 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L9)
```python
import httpx
import openai
import pytest
import torch

from ....utils import RemoteOpenAIServer
from .embed_utils import run_client_embeddings
```
**EN:** Imports third-party packages like `httpx`, `openai`, `pytest`, project helpers such as `....utils.RemoteOpenAIServer`, `.embed_utils.run_client_embeddings`.
**CN:** 导入第三方包（如 `httpx`、`openai`、`pytest`）、项目内辅助模块（如 `....utils.RemoteOpenAIServer`、`.embed_utils.run_client_embeddings`）。

### Module setup / 模块级配置: MODEL_NAME, MAX_MODEL_LEN, sentences_1 (L11-L27)
```python
MODEL_NAME = "BAAI/bge-m3"
MAX_MODEL_LEN = 512


# Example from https://huggingface.co/BAAI/bge-m3
sentences_1 = ["What is BGE M3?", "Definition of BM25"]
sentences_2 = [
    "BGE M3 is an embedding model supporting dense retrieval, "
    "lexical matching and multi-vector interaction.",
    "BM25 is a bag-of-words retrieval function that ranks a set "
    "of documents based on the query terms appearing in each document",
]

similarity_reference = [[0.6259, 0.3474], [0.3309, 0.6734]]
lexical_score_reference = [0.19554901123046875, 0.0]
colbert_score_reference = [0.7797, 0.4620]
SUPPORTED_TASKS = ["embed", "token_embed", "token_classify"]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `MAX_MODEL_LEN`, `sentences_1`, `sentences_2`, `similarity_reference`, `lexical_score_reference`, `colbert_score_reference`, `SUPPORTED_TASKS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`MAX_MODEL_LEN`、`sentences_1`、`sentences_2`、`similarity_reference`、`lexical_score_reference`、`colbert_score_reference`、`SUPPORTED_TASKS`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: pooling_task (L30-L32)
```python
@pytest.fixture(scope="module", params=SUPPORTED_TASKS)
def pooling_task(request):
    yield request.param
```
**EN:** This fixture prepares `pooling_task` for dependent tests. Key inputs are `request`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `pooling_task`。 关键输入包括 `request`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: server (L35-L47)
```python
@pytest.fixture(scope="module")
def server(pooling_task):
    args = [
        "--max-model-len",
        str(MAX_MODEL_LEN),
        "--hf-overrides",
        '{"architectures": ["BgeM3EmbeddingModel"]}',
        "--pooler-config.task",
        pooling_task,
    ]

    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. Key inputs are `pooling_task`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 关键输入包括 `pooling_task`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_bge_m3_api_server_embedding (L50-L81)
```python
@pytest.mark.asyncio
async def test_bge_m3_api_server_embedding(server, pooling_task):
    client = server.get_async_client()

    if pooling_task != "embed":
        with pytest.raises(openai.InternalServerError):
            await run_client_embeddings(
                client,
                MODEL_NAME,
                sentences_1,
            )
        return

    embeddings_list_1 = await run_client_embeddings(
        client,
        MODEL_NAME,
        sentences_1,
    )
# ... 6 lines omitted for brevity ...
    embeddings_1 = torch.tensor(embeddings_list_1)
    embeddings_2 = torch.tensor(embeddings_list_2)
    similarity = embeddings_1 @ embeddings_2.T

    # reference values from BAAI/bge-m3 documentation
    reference = torch.tensor(similarity_reference)

    assert torch.allclose(similarity, reference, rtol=0.01)
```
**EN:** This async test validates `test_bge_m3_api_server_embedding`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `pooling_task`. It checks an expected failure path with `pytest.raises`. The main assertion is `torch.allclose(similarity, reference, rtol=0.01)`.
**CN:** 这个异步测试验证 `test_bge_m3_api_server_embedding`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`pooling_task`。 它使用 `pytest.raises` 检查预期失败路径。 核心断言是 `torch.allclose(similarity, reference, rtol=0.01)`。

### Helper / 辅助函数: tokenize (L84-L94)
```python
async def tokenize(client: openai.AsyncOpenAI, sentences: list[str]) -> list[list[int]]:
    futures = []
    for sentence in sentences:
        futures.append(
            client.post(
                "../tokenize",
                body={"model": MODEL_NAME, "prompt": sentence},
                cast_to=httpx.Response,
            )
        )
    return [(await future).json()["tokens"] for future in futures]
```
**EN:** This async helper encapsulates reusable logic in `tokenize`. Key inputs are `client`, `sentences`. It returns computed state or helper objects back to the caller.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `tokenize` 中。 关键输入包括 `client`、`sentences`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: sparse_embeddings (L97-L117)
```python
async def sparse_embeddings(
    client: openai.AsyncOpenAI, sentences: list[str]
) -> list[dict[int, float]]:
    all_tokens = await tokenize(client, sentences)
    result = await client.post(
        "../pooling",
        body={"model": MODEL_NAME, "input": sentences, "task": "token_classify"},
        cast_to=httpx.Response,
    )
    all_embeddings = [data["data"] for data in result.json()["data"]]

    ret = []

    for sent_tokens, sent_emb in zip(all_tokens, all_embeddings):
        token_embs = dict[int, float]()
        if sent_tokens[0] == 0:
            sent_tokens = sent_tokens[1:]
        for token, val in zip(sent_tokens, sent_emb):
            token_embs[token] = max(val, token_embs.get(token, 0.0))
        ret.append(token_embs)
    return ret
```
**EN:** This async helper encapsulates reusable logic in `sparse_embeddings`. Key inputs are `client`, `sentences`. It returns computed state or helper objects back to the caller.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `sparse_embeddings` 中。 关键输入包括 `client`、`sentences`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: compute_lexical_matching_score (L121-L128)
```python
def compute_lexical_matching_score(
    lw1: dict[int, float], lw2: dict[int, float]
) -> float:
    scores = 0.0
    for token, weight in lw1.items():
        if token in lw2:
            scores += weight * lw2[token]
    return scores
```
**EN:** This helper encapsulates reusable logic in `compute_lexical_matching_score`. Key inputs are `lw1`, `lw2`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `compute_lexical_matching_score` 中。 关键输入包括 `lw1`、`lw2`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_bge_m3_api_server_sparse_embedding (L131-L155)
```python
@pytest.mark.asyncio
async def test_bge_m3_api_server_sparse_embedding(server, pooling_task):
    client = server.get_async_client()

    if pooling_task != "token_classify":
        with pytest.raises(openai.BadRequestError):
            await sparse_embeddings(client, sentences_1)
        return

    embeddings_1 = await sparse_embeddings(client, sentences_1)
    embeddings_2 = await sparse_embeddings(client, sentences_2)

    lexical_scores_1_0_x_2_0 = compute_lexical_matching_score(
        embeddings_1[0], embeddings_2[0]
    )
    assert lexical_scores_1_0_x_2_0 == pytest.approx(
        lexical_score_reference[0], rel=0.01
    )

    lexical_scores_1_0_x_1_1 = compute_lexical_matching_score(
        embeddings_1[0], embeddings_1[1]
    )
    assert lexical_scores_1_0_x_1_1 == pytest.approx(
        lexical_score_reference[1], rel=0.01
    )
```
**EN:** This async test validates `test_bge_m3_api_server_sparse_embedding`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `pooling_task`. It checks an expected failure path with `pytest.raises`. The main assertion is `lexical_scores_1_0_x_2_0 == pytest.approx(lexical_score_reference[0], rel=0.01)` and `lexical_scores_1_0_x_1_1 == pytest.approx(lexical_score_reference[1], rel=0.01)`.
**CN:** 这个异步测试验证 `test_bge_m3_api_server_sparse_embedding`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`pooling_task`。 它使用 `pytest.raises` 检查预期失败路径。 核心断言是 `lexical_scores_1_0_x_2_0 == pytest.approx(lexical_score_reference[0], rel=0.01)` and `lexical_scores_1_0_x_1_1 == pytest.approx(lexical_score_reference[1], rel=0.01)`。

### Test / 测试: test_bge_m3_api_server_sparse_embedding_corner_case (L158-L167)
```python
@pytest.mark.asyncio
async def test_bge_m3_api_server_sparse_embedding_corner_case(server, pooling_task):
    if pooling_task != "token_classify":
        return

    client = server.get_async_client()
    embeddings = await sparse_embeddings(client, ["Hi"])
    assert len(embeddings) == 1
    assert 2673 in embeddings[0]
    assert embeddings[0][2673] == pytest.approx(0.26710861921310425, rel=0.01)
```
**EN:** This async test validates `test_bge_m3_api_server_sparse_embedding_corner_case`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `pooling_task`. The main assertion is `len(embeddings) == 1` and `2673 in embeddings[0]`.
**CN:** 这个异步测试验证 `test_bge_m3_api_server_sparse_embedding_corner_case`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`pooling_task`。 核心断言是 `len(embeddings) == 1` and `2673 in embeddings[0]`。

### Helper / 辅助函数: colbert_score (L171-L175)
```python
def colbert_score(q_reps: torch.Tensor, p_reps: torch.Tensor) -> torch.Tensor:
    token_scores = torch.einsum("in,jn->ij", q_reps, p_reps)
    scores, _ = token_scores.max(-1)
    scores = torch.sum(scores) / q_reps.size(0)
    return scores
```
**EN:** This helper encapsulates reusable logic in `colbert_score`. Key inputs are `q_reps`, `p_reps`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `colbert_score` 中。 关键输入包括 `q_reps`、`p_reps`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_bge_m3_api_server_multi_vector (L178-L212)
```python
@pytest.mark.asyncio
async def test_bge_m3_api_server_multi_vector(server, pooling_task):
    client = server.get_async_client()

    if pooling_task != "token_embed":
        with pytest.raises(openai.BadRequestError):
            await client.post(
                "../pooling",
                body={"model": MODEL_NAME, "input": sentences_1, "task": "token_embed"},
                cast_to=httpx.Response,
            )
        return

    result_1 = await client.post(
        "../pooling",
        body={"model": MODEL_NAME, "input": sentences_1, "task": "token_embed"},
        cast_to=httpx.Response,
    )
# ... 9 lines omitted for brevity ...
    colbert_score_1_0_x_2_0 = colbert_score(embeddings_1[0], embeddings_2[0])
    assert colbert_score_1_0_x_2_0 == pytest.approx(
        colbert_score_reference[0], rel=0.01
    )
    colbert_score_1_0_x_2_1 = colbert_score(embeddings_1[0], embeddings_2[1])
    assert colbert_score_1_0_x_2_1 == pytest.approx(
        colbert_score_reference[1], rel=0.01
    )
```
**EN:** This async test validates `test_bge_m3_api_server_multi_vector`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `pooling_task`. It checks an expected failure path with `pytest.raises`. The main assertion is `colbert_score_1_0_x_2_0 == pytest.approx(colbert_score_reference[0], rel=0.01)` and `colbert_score_1_0_x_2_1 == pytest.approx(colbert_score_reference[1], rel=0.01)`.
**CN:** 这个异步测试验证 `test_bge_m3_api_server_multi_vector`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`pooling_task`。 它使用 `pytest.raises` 检查预期失败路径。 核心断言是 `colbert_score_1_0_x_2_0 == pytest.approx(colbert_score_reference[0], rel=0.01)` and `colbert_score_1_0_x_2_1 == pytest.approx(colbert_score_reference[1], rel=0.01)`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `httpx`, `openai`, `pytest`, `torch`
- **Local relative imports / 本地相对导入**: `....utils.RemoteOpenAIServer`, `.embed_utils.run_client_embeddings`
