# test_cross_encoder_online_vision.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/scoring/test_cross_encoder_online_vision.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and scoring or reranking behavior. The file defines 11 test(s), 1 fixture(s), and 3 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与打分或重排行为。它定义了 11 个测试、1 个 fixture，以及 3 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L12)
```python
import json

import pytest
import requests

from tests.utils import VLLM_PATH, RemoteOpenAIServer
from vllm.entrypoints.pooling.scoring.protocol import RerankResponse, ScoreResponse
from vllm.multimodal.utils import encode_image_url, fetch_image
from vllm.platforms import current_platform
```
**EN:** Imports standard-library modules such as `json`, third-party packages like `pytest`, `requests`, project helpers such as `tests.utils.RemoteOpenAIServer`, `tests.utils.VLLM_PATH`, `vllm.entrypoints.pooling.scoring.protocol.RerankResponse`.
**CN:** 导入标准库模块（如 `json`）、第三方包（如 `pytest`、`requests`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`tests.utils.VLLM_PATH`、`vllm.entrypoints.pooling.scoring.protocol.RerankResponse`）。

### Module setup / 模块级配置: MODEL_NAME, HF_OVERRIDES, ROCM_ATTN_BACKENDS (L14-L62)
```python
MODEL_NAME = "Qwen/Qwen3-VL-Reranker-2B"
HF_OVERRIDES = {
    "architectures": ["Qwen3VLForSequenceClassification"],
    "classifier_from_token": ["no", "yes"],
    "is_original_qwen3_reranker": True,
}

ROCM_ATTN_BACKENDS = [
    "ROCM_ATTN",
    "ROCM_AITER_FA",
    "TRITON_ATTN",
    "FLEX_ATTENTION",
]

ATTN_BACKENDS = ROCM_ATTN_BACKENDS if current_platform.is_rocm() else ["auto"]

# ... 27 lines omitted for brevity ...
# test flakiness.
ROCM_EXTRA_ARGS = (
    ["--no-enable-prefix-caching", "--max-num-seqs", "1"]
    if current_platform.is_rocm()
    else []
)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `HF_OVERRIDES`, `ROCM_ATTN_BACKENDS`, `ATTN_BACKENDS`, `BACKEND_TOL`, `BACKEND_ABS_TOL`, `ROCM_ENV_OVERRIDES`, `ROCM_EXTRA_ARGS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`HF_OVERRIDES`、`ROCM_ATTN_BACKENDS`、`ATTN_BACKENDS`、`BACKEND_TOL`、`BACKEND_ABS_TOL`、`ROCM_ENV_OVERRIDES`、`ROCM_EXTRA_ARGS`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: get_tol (L65-L66)
```python
def get_tol(backend: str) -> float:
    return BACKEND_TOL.get(backend, BACKEND_TOL["default"])
```
**EN:** This helper encapsulates reusable logic in `get_tol`. Key inputs are `backend`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_tol` 中。 关键输入包括 `backend`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: get_abs_tol (L69-L70)
```python
def get_abs_tol(backend: str) -> float:
    return BACKEND_ABS_TOL.get(backend, BACKEND_ABS_TOL["default"])
```
**EN:** This helper encapsulates reusable logic in `get_abs_tol`. Key inputs are `backend`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_abs_tol` 中。 关键输入包括 `backend`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: assert_score (L73-L86)
```python
def assert_score(actual: float, expected: float, backend: str, label: str):
    tol = get_tol(backend)
    abs_tol = get_abs_tol(backend)
    diff = abs(actual - expected)
    rel_diff = diff / abs(expected) if expected != 0 else diff
    print(
        f"[{backend}] {label}: actual={actual:.6f} expected={expected:.6f} "
        f"diff={diff:.6f} rel_diff={rel_diff:.4f} tol={tol} abs_tol={abs_tol}"
    )
    assert actual == pytest.approx(expected, rel=tol, abs=abs_tol), (
        f"[{backend}] {label}: score mismatch — "
        f"actual={actual:.6f}, expected={expected:.6f}, "
        f"rel_diff={rel_diff:.4f}, tol={tol}, abs_tol={abs_tol}"
    )
```
**EN:** This helper encapsulates reusable logic in `assert_score`. Key inputs are `actual`, `expected`, `backend`, `label`. The main assertion is `actual == pytest.approx(expected, rel=tol, abs=abs_tol)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `assert_score` 中。 关键输入包括 `actual`、`expected`、`backend`、`label`。 核心断言是 `actual == pytest.approx(expected, rel=tol, abs=abs_tol)`。

### Module setup / 模块级配置: query, document, image_url (L89-L109)
```python
query = "A cat standing in the snow."
document = "This product was excellent and exceeded my expectations."
image_url = "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/cat_snow.jpg"
documents = [
    {
        "type": "text",
        "text": document,
    },
    {
        "type": "image_url",
        "image_url": {"url": image_url},
    },
    {
        "type": "image_url",
        "image_url": {"url": encode_image_url(fetch_image(image_url))},
    },
]

TEXT_VS_TEXT = 0.10040374100208282
TEXT_VS_IMAGE = 0.7423753142356873
TEXT_VS_TEXT_PLUS_IMAGE = 0.5298863053321838
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `query`, `document`, `image_url`, `documents`, `TEXT_VS_TEXT`, `TEXT_VS_IMAGE`, `TEXT_VS_TEXT_PLUS_IMAGE`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `query`、`document`、`image_url`、`documents`、`TEXT_VS_TEXT`、`TEXT_VS_IMAGE`、`TEXT_VS_TEXT_PLUS_IMAGE`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L112-L137)
```python
@pytest.fixture(scope="module", params=ATTN_BACKENDS)
def server(request):
    backend = request.param
    print(f"\n=== Starting server with attention backend: {backend} ===")
    args = [
        "--enforce-eager",
        "--max-model-len",
        "8192",
        "--chat-template",
        str(VLLM_PATH / "examples/pooling/score/template/qwen3_vl_reranker.jinja"),
    ]

    env = dict()
    if backend != "auto":
        args += ["--attention-config", json.dumps({"backend": backend})]
        args += ROCM_EXTRA_ARGS

        env = dict(ROCM_ENV_OVERRIDES)
        if backend != "ROCM_AITER_FA":
            env["VLLM_ROCM_USE_AITER"] = "0"

    with RemoteOpenAIServer(
        MODEL_NAME, args, override_hf_configs=HF_OVERRIDES, env_dict=env
    ) as remote_server:
        print(f"=== Server ready with backend: {backend} ===")
        yield remote_server, backend
```
**EN:** This fixture prepares `server` for dependent tests. Key inputs are `request`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 关键输入包括 `request`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_score_api_queries_str_documents_str (L140-L160)
```python
@pytest.mark.asyncio
async def test_score_api_queries_str_documents_str(
    server: tuple[RemoteOpenAIServer, str],
):
    remote_server, backend = server
    score_response = requests.post(
        remote_server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "queries": query,
            "documents": document,
        },
    )
    score_response.raise_for_status()
    score = ScoreResponse.model_validate(score_response.json())

    assert score.id is not None
    assert score.data is not None
    assert len(score.data) == 1
    assert score.usage.prompt_tokens == 81
    assert_score(score.data[0].score, TEXT_VS_TEXT, backend, "text_vs_text")
```
**EN:** This async test validates `test_score_api_queries_str_documents_str`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `score.id is not None` and `score.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_queries_str_documents_str`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `score.id is not None` and `score.data is not None`。

### Test / 测试: test_score_api_queries_str_documents_text_content (L163-L183)
```python
@pytest.mark.asyncio
async def test_score_api_queries_str_documents_text_content(
    server: tuple[RemoteOpenAIServer, str],
):
    remote_server, backend = server
    score_response = requests.post(
        remote_server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "queries": query,
            "documents": {"content": [documents[0]]},
        },
    )
    score_response.raise_for_status()
    score = ScoreResponse.model_validate(score_response.json())

    assert score.id is not None
    assert score.data is not None
    assert len(score.data) == 1
    assert score.usage.prompt_tokens == 81
    assert_score(score.data[0].score, TEXT_VS_TEXT, backend, "text_vs_text")
```
**EN:** This async test validates `test_score_api_queries_str_documents_text_content`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `score.id is not None` and `score.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_queries_str_documents_text_content`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `score.id is not None` and `score.data is not None`。

### Test / 测试: test_score_api_queries_str_documents_image_url_content (L186-L206)
```python
@pytest.mark.asyncio
async def test_score_api_queries_str_documents_image_url_content(
    server: tuple[RemoteOpenAIServer, str],
):
    remote_server, backend = server
    score_response = requests.post(
        remote_server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "queries": query,
            "documents": {"content": [documents[1]]},
        },
    )
    score_response.raise_for_status()
    score = ScoreResponse.model_validate(score_response.json())

    assert score.id is not None
    assert score.data is not None
    assert len(score.data) == 1
    assert score.usage.prompt_tokens == 98
    assert_score(score.data[0].score, TEXT_VS_IMAGE, backend, "text_vs_image")
```
**EN:** This async test validates `test_score_api_queries_str_documents_image_url_content`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `score.id is not None` and `score.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_queries_str_documents_image_url_content`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `score.id is not None` and `score.data is not None`。

### Test / 测试: test_score_api_queries_str_documents_image_base64_content (L209-L229)
```python
@pytest.mark.asyncio
async def test_score_api_queries_str_documents_image_base64_content(
    server: tuple[RemoteOpenAIServer, str],
):
    remote_server, backend = server
    score_response = requests.post(
        remote_server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "queries": query,
            "documents": {"content": [documents[2]]},
        },
    )
    score_response.raise_for_status()
    score = ScoreResponse.model_validate(score_response.json())

    assert score.id is not None
    assert score.data is not None
    assert len(score.data) == 1
    assert score.usage.prompt_tokens == 98
    assert_score(score.data[0].score, TEXT_VS_IMAGE, backend, "text_vs_image_base64")
```
**EN:** This async test validates `test_score_api_queries_str_documents_image_base64_content`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `score.id is not None` and `score.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_queries_str_documents_image_base64_content`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `score.id is not None` and `score.data is not None`。

### Test / 测试: test_score_api_queries_str_documents_image_url_plus_text_content (L232-L254)
```python
@pytest.mark.asyncio
async def test_score_api_queries_str_documents_image_url_plus_text_content(
    server: tuple[RemoteOpenAIServer, str],
):
    remote_server, backend = server
    score_response = requests.post(
        remote_server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "queries": query,
            "documents": {"content": [documents[0], documents[1]]},
        },
    )
    score_response.raise_for_status()
    score = ScoreResponse.model_validate(score_response.json())

    assert score.id is not None
    assert score.data is not None
    assert len(score.data) == 1
    assert score.usage.prompt_tokens == 107
    assert_score(
        score.data[0].score, TEXT_VS_TEXT_PLUS_IMAGE, backend, "text_vs_text_plus_image"
    )
```
**EN:** This async test validates `test_score_api_queries_str_documents_image_url_plus_text_content`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `score.id is not None` and `score.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_queries_str_documents_image_url_plus_text_content`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `score.id is not None` and `score.data is not None`。

### Test / 测试: test_score_api_queries_str_documents_list (L257-L290)
```python
@pytest.mark.asyncio
async def test_score_api_queries_str_documents_list(
    server: tuple[RemoteOpenAIServer, str],
):
    remote_server, backend = server
    score_response = requests.post(
        remote_server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "queries": query,
            "documents": [
                document,
                {"content": [documents[0]]},
                {"content": [documents[1]]},
                {"content": [documents[0], documents[1]]},
            ],
        },
    )
# ... 8 lines omitted for brevity ...
    assert_score(score.data[1].score, TEXT_VS_TEXT, backend, "list[1]_text_vs_text")
    assert_score(score.data[2].score, TEXT_VS_IMAGE, backend, "list[2]_text_vs_image")
    assert_score(
        score.data[3].score,
        TEXT_VS_TEXT_PLUS_IMAGE,
        backend,
        "list[3]_text_vs_text_plus_image",
    )
```
**EN:** This async test validates `test_score_api_queries_str_documents_list`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `score.id is not None` and `score.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_queries_str_documents_list`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `score.id is not None` and `score.data is not None`。

### Test / 测试: test_rerank_api_queries_str_documents_list (L293-L343)
```python
@pytest.mark.asyncio
async def test_rerank_api_queries_str_documents_list(
    server: tuple[RemoteOpenAIServer, str],
):
    remote_server, backend = server
    rerank_response = requests.post(
        remote_server.url_for("rerank"),
        json={
            "model": MODEL_NAME,
            "query": query,
            "documents": [
                document,
                {"content": [documents[0]]},
                {"content": [documents[1]]},
                {"content": [documents[0], documents[1]]},
            ],
        },
    )
# ... 25 lines omitted for brevity ...
        "rerank[2]_text_vs_image",
    )
    assert_score(
        rerank.results[3].relevance_score,
        TEXT_VS_TEXT_PLUS_IMAGE,
        backend,
        "rerank[3]_text_vs_text_plus_image",
    )
```
**EN:** This async test validates `test_rerank_api_queries_str_documents_list`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `rerank.id is not None` and `rerank.model is not None`.
**CN:** 这个异步测试验证 `test_rerank_api_queries_str_documents_list`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `rerank.id is not None` and `rerank.model is not None`。

### Test / 测试: test_score_api_queries_list_documents_list (L346-L379)
```python
@pytest.mark.asyncio
async def test_score_api_queries_list_documents_list(
    server: tuple[RemoteOpenAIServer, str],
):
    remote_server, backend = server
    score_response = requests.post(
        remote_server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "queries": [query] * 4,
            "documents": [
                document,
                {"content": [documents[0]]},
                {"content": [documents[1]]},
                {"content": [documents[0], documents[1]]},
            ],
        },
    )
# ... 8 lines omitted for brevity ...
    assert_score(score.data[1].score, TEXT_VS_TEXT, backend, "paired[1]_text_vs_text")
    assert_score(score.data[2].score, TEXT_VS_IMAGE, backend, "paired[2]_text_vs_image")
    assert_score(
        score.data[3].score,
        TEXT_VS_TEXT_PLUS_IMAGE,
        backend,
        "paired[3]_text_vs_text_plus_image",
    )
```
**EN:** This async test validates `test_score_api_queries_list_documents_list`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `score.id is not None` and `score.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_queries_list_documents_list`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `score.id is not None` and `score.data is not None`。

### Module setup / 模块级配置: INSTRUCTION (L382-L385)
```python
INSTRUCTION = (
    "Given a multimodal retrieval query, retrieve candidates that "
    "visually or textually match the requested scene, object, or action."
)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `INSTRUCTION`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `INSTRUCTION`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_score_api_instruction_field (L388-L420)
```python
@pytest.mark.asyncio
async def test_score_api_instruction_field(
    server: tuple[RemoteOpenAIServer, str],
):
    remote_server, _ = server

    default_response = requests.post(
        remote_server.url_for("score"),
        json={
            "model": MODEL_NAME,
            "queries": query,
            "documents": document,
        },
    )
    default_response.raise_for_status()
    default_score = ScoreResponse.model_validate(default_response.json())

    instruction_response = requests.post(
# ... 7 lines omitted for brevity ...
    )
    instruction_response.raise_for_status()
    instruction_score = ScoreResponse.model_validate(instruction_response.json())

    assert instruction_score.id is not None
    assert instruction_score.data is not None
    assert len(instruction_score.data) == 1
    assert instruction_score.usage.prompt_tokens > default_score.usage.prompt_tokens
```
**EN:** This async test validates `test_score_api_instruction_field`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `instruction_score.id is not None` and `instruction_score.data is not None`.
**CN:** 这个异步测试验证 `test_score_api_instruction_field`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `instruction_score.id is not None` and `instruction_score.data is not None`。

### Test / 测试: test_rerank_api_instruction_field (L423-L463)
```python
@pytest.mark.asyncio
async def test_rerank_api_instruction_field(
    server: tuple[RemoteOpenAIServer, str],
):
    remote_server, _ = server

    doc_list = [
        document,
        {"content": [documents[0]]},
        {"content": [documents[1]]},
        {"content": [documents[0], documents[1]]},
    ]

    default_response = requests.post(
        remote_server.url_for("rerank"),
        json={
            "model": MODEL_NAME,
            "query": query,
# ... 15 lines omitted for brevity ...
    instruction_response.raise_for_status()
    instruction_rerank = RerankResponse.model_validate(instruction_response.json())

    assert instruction_rerank.id is not None
    assert instruction_rerank.model is not None
    assert instruction_rerank.usage is not None
    assert len(instruction_rerank.results) == len(default_rerank.results)
    assert instruction_rerank.usage.prompt_tokens > default_rerank.usage.prompt_tokens
```
**EN:** This async test validates `test_rerank_api_instruction_field`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `instruction_rerank.id is not None` and `instruction_rerank.model is not None`.
**CN:** 这个异步测试验证 `test_rerank_api_instruction_field`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `instruction_rerank.id is not None` and `instruction_rerank.model is not None`。

### Test / 测试: test_rerank_api_instruction_field_matches_chat_template_kwargs (L466-L511)
```python
@pytest.mark.asyncio
async def test_rerank_api_instruction_field_matches_chat_template_kwargs(
    server: tuple[RemoteOpenAIServer, str],
):
    remote_server, _ = server

    doc_list = [
        document,
        {"content": [documents[0]]},
        {"content": [documents[1]]},
        {"content": [documents[0], documents[1]]},
    ]

    field_response = requests.post(
        remote_server.url_for("rerank"),
        json={
            "model": MODEL_NAME,
            "query": query,
# ... 20 lines omitted for brevity ...

    field_scores = [
        r.relevance_score for r in sorted(field_rerank.results, key=lambda x: x.index)
    ]
    kwargs_scores = [
        r.relevance_score for r in sorted(kwargs_rerank.results, key=lambda x: x.index)
    ]
    assert field_scores == pytest.approx(kwargs_scores)
```
**EN:** This async test validates `test_rerank_api_instruction_field_matches_chat_template_kwargs`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `kwargs_rerank.usage.prompt_tokens == field_rerank.usage.prompt_tokens` and `field_scores == pytest.approx(kwargs_scores)`.
**CN:** 这个异步测试验证 `test_rerank_api_instruction_field_matches_chat_template_kwargs`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `kwargs_rerank.usage.prompt_tokens == field_rerank.usage.prompt_tokens` and `field_scores == pytest.approx(kwargs_scores)`。

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
- **Stdlib / 标准库**: `json`
- **Third-party / 第三方**: `pytest`, `requests`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `tests.utils.VLLM_PATH`, `vllm.entrypoints.pooling.scoring.protocol.RerankResponse`, `vllm.entrypoints.pooling.scoring.protocol.ScoreResponse`, `vllm.multimodal.utils.encode_image_url`, `vllm.multimodal.utils.fetch_image`, `vllm.platforms.current_platform`
