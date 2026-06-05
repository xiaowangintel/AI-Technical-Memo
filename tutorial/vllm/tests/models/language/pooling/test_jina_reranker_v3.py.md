# test_jina_reranker_v3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/pooling/test_jina_reranker_v3.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and language-model behavior. The file defines 2 test(s), 0 fixture(s), and 10 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与语言模型行为。它定义了 2 个测试、0 个 fixture，以及 10 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L11)
```python
import pytest
import requests
import torch
import torch.nn.functional as F

from tests.utils import RemoteOpenAIServer
from vllm.entrypoints.pooling.pooling.protocol import PoolingResponse
from vllm.entrypoints.pooling.scoring.protocol import ScoreResponse
```
**EN:** Imports third-party packages like `pytest`, `requests`, `torch`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.pooling.pooling.protocol.PoolingResponse`, `vllm.entrypoints.pooling.scoring.protocol.ScoreResponse`.
**CN:** 导入第三方包（如 `pytest`、`requests`、`torch`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.entrypoints.pooling.pooling.protocol.PoolingResponse`、`vllm.entrypoints.pooling.scoring.protocol.ScoreResponse`）。

### Module setup / 模块级配置: model_name, query, documents (L13-L41)
```python
model_name = "jinaai/jina-reranker-v3"
query = "What are the health benefits of green tea?"
documents = [
    "Green tea contains antioxidants called catechins that may help reduce inflammation and protect cells from damage.",
    "El precio del café ha aumentado un 20% este año debido a problemas en la cadena de suministro.",
    "Studies show that drinking green tea regularly can improve brain function and boost metabolism.",
    "Basketball is one of the most popular sports in the United States.",
    "绿茶富含儿茶素等抗氧化剂，可以降低心脏病风险，还有助于控制体重。",
    "Le thé vert est riche en antioxydants et peut améliorer la fonction cérébrale.",
]

EMBEDDING_SIZE = 512
REFERENCE_1_VS_1 = [
    0.345703125,
    -0.10498046,
    0.314453125,
# ... 7 lines omitted for brevity ...
    0.189453125,
    -0.1708984375,
    0.2255859375,
    0.1640625,
]
TOL = 0.01
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `model_name`, `query`, `documents`, `EMBEDDING_SIZE`, `REFERENCE_1_VS_1`, `REFERENCE_1_VS_N`, `TOL`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `model_name`、`query`、`documents`、`EMBEDDING_SIZE`、`REFERENCE_1_VS_1`、`REFERENCE_1_VS_N`、`TOL`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_offline (L44-L51)
```python
def test_offline(vllm_runner):
    with vllm_runner(model_name, runner="pooling") as llm_runner:
        llm = llm_runner.get_llm()
        _test_offline_1_v_1(llm)
        _test_offline_1_v_n(llm)
        _test_offline_n_v_n(llm)
        _test_offline_token_embed_illegal_inputs(llm)
        assert llm.model_config.embedding_size == EMBEDDING_SIZE
```
**EN:** This test validates `test_offline`. Key inputs are `vllm_runner`. The main assertion is `llm.model_config.embedding_size == EMBEDDING_SIZE`.
**CN:** 这个测试验证 `test_offline`。 关键输入包括 `vllm_runner`。 核心断言是 `llm.model_config.embedding_size == EMBEDDING_SIZE`。

### Test / 测试: test_online (L54-L59)
```python
def test_online():
    with RemoteOpenAIServer(model_name, ["--runner", "pooling"]) as server:
        _test_online_1_v_1(server)
        _test_online_1_v_n(server)
        _test_online_n_v_n(server)
        _test_online_token_embed_illegal_inputs(server)
```
**EN:** This test validates `test_online`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint.
**CN:** 这个测试验证 `test_online`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。

### Helper / 辅助函数: _test_offline_1_v_1 (L62-L78)
```python
def _test_offline_1_v_1(llm):
    # test llm.score
    outputs = llm.score(query, documents[0])
    assert len(outputs) == 1
    assert outputs[0].outputs.score == pytest.approx(REFERENCE_1_VS_1[0], abs=TOL)

    # test llm.encode
    outputs = llm.encode(documents[:1] + [query], pooling_task="token_embed")
    embeds = outputs[0].outputs.data.float()
    assert embeds.shape[0] == 2
    assert embeds.shape[-1] == EMBEDDING_SIZE

    doc_embeds = embeds[:-1]
    query_embeds = embeds[-1]

    scores = F.cosine_similarity(query_embeds, doc_embeds)
    assert scores[0] == pytest.approx(REFERENCE_1_VS_1[0], abs=TOL)
```
**EN:** This helper encapsulates reusable logic in `_test_offline_1_v_1`. Key inputs are `llm`. The main assertion is `len(outputs) == 1` and `outputs[0].outputs.score == pytest.approx(REFERENCE_1_VS_1[0], abs=TOL)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_test_offline_1_v_1` 中。 关键输入包括 `llm`。 核心断言是 `len(outputs) == 1` and `outputs[0].outputs.score == pytest.approx(REFERENCE_1_VS_1[0], abs=TOL)`。

### Helper / 辅助函数: _test_offline_1_v_n (L81-L102)
```python
def _test_offline_1_v_n(llm):
    # test llm.score
    outputs = llm.score(query, documents)
    assert len(outputs) == len(documents)

    for expected, output in zip(REFERENCE_1_VS_N, outputs):
        actual = output.outputs.score
        assert actual == pytest.approx(expected, abs=TOL)

    # test llm.encode
    outputs = llm.encode(documents + [query], pooling_task="token_embed")
    embeds = outputs[0].outputs.data.float()
    assert embeds.shape[0] == len(documents) + 1

    doc_embeds = embeds[:-1]
    query_embeds = embeds[-1]

    scores = F.cosine_similarity(query_embeds, doc_embeds)

    assert len(scores) == len(documents)
    for expected, actual in zip(REFERENCE_1_VS_N, scores):
        assert actual == pytest.approx(expected, abs=TOL)
```
**EN:** This helper encapsulates reusable logic in `_test_offline_1_v_n`. Key inputs are `llm`. The main assertion is `len(outputs) == len(documents)` and `embeds.shape[0] == len(documents) + 1`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_test_offline_1_v_n` 中。 关键输入包括 `llm`。 核心断言是 `len(outputs) == len(documents)` and `embeds.shape[0] == len(documents) + 1`。

### Helper / 辅助函数: _test_offline_n_v_n (L105-L124)
```python
def _test_offline_n_v_n(llm):
    # test llm.score
    outputs = llm.score([query] * len(documents), documents)
    assert len(outputs) == len(documents)

    for expected, output in zip(REFERENCE_1_VS_1, outputs):
        actual = output.outputs.score
        assert actual == pytest.approx(expected, abs=TOL)

    # test llm.encode
    for doc, expected in zip(documents, REFERENCE_1_VS_1):
        outputs = llm.encode([doc, query], pooling_task="token_embed")
        embeds = outputs[0].outputs.data.float()
        assert embeds.shape[0] == 2

        doc_embeds = embeds[:-1]
        query_embeds = embeds[-1]

        scores = F.cosine_similarity(query_embeds, doc_embeds)
        assert scores[0] == pytest.approx(expected, abs=TOL)
```
**EN:** This helper encapsulates reusable logic in `_test_offline_n_v_n`. Key inputs are `llm`. The main assertion is `len(outputs) == len(documents)` and `actual == pytest.approx(expected, abs=TOL)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_test_offline_n_v_n` 中。 关键输入包括 `llm`。 核心断言是 `len(outputs) == len(documents)` and `actual == pytest.approx(expected, abs=TOL)`。

### Helper / 辅助函数: _test_offline_token_embed_illegal_inputs (L127-L136)
```python
def _test_offline_token_embed_illegal_inputs(llm):
    with pytest.raises(
        ValueError, match="The JinaForRanking model requires at least 2 inputs."
    ):
        llm.encode([query], pooling_task="token_embed")

    with pytest.raises(
        ValueError, match="The JinaForRanking model only supports text as input."
    ):
        llm.encode([1, 2, 3], pooling_task="token_embed")
```
**EN:** This helper encapsulates reusable logic in `_test_offline_token_embed_illegal_inputs`. Key inputs are `llm`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_test_offline_token_embed_illegal_inputs` 中。 关键输入包括 `llm`。 它使用 `pytest.raises` 检查预期失败路径。

### Helper / 辅助函数: _get_scores (L139-L152)
```python
def _get_scores(server, query, document):
    score_response = requests.post(
        server.url_for("score"),
        json={
            "model": model_name,
            "queries": query,
            "documents": document,
        },
    )

    score_response.raise_for_status()
    score = ScoreResponse.model_validate(score_response.json())

    return [d.score for d in score.data]
```
**EN:** This helper encapsulates reusable logic in `_get_scores`. Key inputs are `server`, `query`, `document`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_get_scores` 中。 关键输入包括 `server`、`query`、`document`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _get_embeds (L155-L168)
```python
def _get_embeds(server, prompts: list[str]):
    response = requests.post(
        server.url_for("pooling"),
        json={
            "model": model_name,
            "task": "token_embed",
            "input": prompts,
            "encoding_format": "float",
        },
    )
    response.raise_for_status()
    poolings = PoolingResponse.model_validate(response.json())

    return torch.as_tensor([d.data for d in poolings.data][0]).float()
```
**EN:** This helper encapsulates reusable logic in `_get_embeds`. Key inputs are `server`, `prompts`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_get_embeds` 中。 关键输入包括 `server`、`prompts`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _test_online_1_v_1 (L171-L186)
```python
def _test_online_1_v_1(server):
    # test scoring api
    scores = _get_scores(server, query, documents[0])
    assert len(scores) == 1
    assert scores[0] == pytest.approx(REFERENCE_1_VS_1[0], abs=TOL)

    # test pooling api
    embeds = _get_embeds(server, [documents[0], query])
    assert embeds.shape[0] == 2
    assert embeds.shape[-1] == EMBEDDING_SIZE

    doc_embeds = embeds[:-1]
    query_embeds = embeds[-1]

    scores = F.cosine_similarity(query_embeds, doc_embeds)
    assert scores[0] == pytest.approx(REFERENCE_1_VS_1[0], abs=TOL)
```
**EN:** This helper encapsulates reusable logic in `_test_online_1_v_1`. Key inputs are `server`. The main assertion is `len(scores) == 1` and `scores[0] == pytest.approx(REFERENCE_1_VS_1[0], abs=TOL)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_test_online_1_v_1` 中。 关键输入包括 `server`。 核心断言是 `len(scores) == 1` and `scores[0] == pytest.approx(REFERENCE_1_VS_1[0], abs=TOL)`。

### Helper / 辅助函数: _test_online_1_v_n (L189-L208)
```python
def _test_online_1_v_n(server):
    # test scoring api
    scores = _get_scores(server, query, documents)
    assert len(scores) == len(documents)

    for expected, actual in zip(REFERENCE_1_VS_N, scores):
        assert actual == pytest.approx(expected, abs=TOL)

    # test pooling api
    embeds = _get_embeds(server, documents + [query])
    assert embeds.shape[0] == len(documents) + 1

    doc_embeds = embeds[:-1]
    query_embeds = embeds[-1]

    scores = F.cosine_similarity(query_embeds, doc_embeds)

    assert len(scores) == len(documents)
    for expected, actual in zip(REFERENCE_1_VS_N, scores):
        assert actual == pytest.approx(expected, abs=TOL)
```
**EN:** This helper encapsulates reusable logic in `_test_online_1_v_n`. Key inputs are `server`. The main assertion is `len(scores) == len(documents)` and `embeds.shape[0] == len(documents) + 1`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_test_online_1_v_n` 中。 关键输入包括 `server`。 核心断言是 `len(scores) == len(documents)` and `embeds.shape[0] == len(documents) + 1`。

### Helper / 辅助函数: _test_online_n_v_n (L211-L229)
```python
def _test_online_n_v_n(server):
    # test scoring api
    scores = _get_scores(server, [query] * len(documents), documents)
    assert len(scores) == len(documents)

    for expected, actual in zip(REFERENCE_1_VS_1, scores):
        assert actual == pytest.approx(expected, abs=TOL)

    # test pooling api
    for doc, expected in zip(documents, REFERENCE_1_VS_1):
        embeds = _get_embeds(server, [doc, query])
        assert embeds.shape[0] == 2

        doc_embeds = embeds[:-1]
        query_embeds = embeds[-1]

        scores = F.cosine_similarity(query_embeds, doc_embeds)
        assert len(scores) == 1
        assert scores[0] == pytest.approx(expected, abs=TOL)
```
**EN:** This helper encapsulates reusable logic in `_test_online_n_v_n`. Key inputs are `server`. The main assertion is `len(scores) == len(documents)` and `actual == pytest.approx(expected, abs=TOL)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_test_online_n_v_n` 中。 关键输入包括 `server`。 核心断言是 `len(scores) == len(documents)` and `actual == pytest.approx(expected, abs=TOL)`。

### Helper / 辅助函数: _test_online_token_embed_illegal_inputs (L232-L275)
```python
def _test_online_token_embed_illegal_inputs(server):
    response = requests.post(
        server.url_for("pooling"),
        json={
            "model": model_name,
            "task": "token_embed",
            "input": [query],
            "encoding_format": "float",
        },
    )
    assert response.json()["error"]["message"].startswith(
        "The JinaForRanking model requires at least 2 inputs."
    )

    response = requests.post(
        server.url_for("pooling"),
        json={
            "model": model_name,
# ... 18 lines omitted for brevity ...
                }
            ],
            "encoding_format": "float",
        },
    )
    assert response.json()["error"]["message"].startswith(
        "The JinaForRanking does not support chat Request."
    )
```
**EN:** This helper encapsulates reusable logic in `_test_online_token_embed_illegal_inputs`. Key inputs are `server`. The main assertion is `response.json()['error']['message'].startswith('The JinaForRanking model requires at least 2 inputs.')` and `response.json()['error']['message'].startswith('The JinaForRanking model only supports text as input.')`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_test_online_token_embed_illegal_inputs` 中。 关键输入包括 `server`。 核心断言是 `response.json()['error']['message'].startswith('The JinaForRanking model requires at least 2 inputs.')` and `response.json()['error']['message'].startswith('The JinaForRanking model only supports text as input.')`。

## Key Concepts / 关键概念
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `pytest`, `requests`, `torch`, `torch.nn.functional`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.pooling.pooling.protocol.PoolingResponse`, `vllm.entrypoints.pooling.scoring.protocol.ScoreResponse`
