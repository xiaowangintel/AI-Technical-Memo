# test_cohere_online.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/embed/test_cohere_online.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior. The file defines 15 test(s), 3 fixture(s), and 4 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为。它定义了 15 个测试、3 个 fixture，以及 4 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L10-L17)
```python
import struct

import numpy as np
import pybase64 as base64
import pytest
import requests

from tests.utils import RemoteOpenAIServer
```
**EN:** Imports standard-library modules such as `struct`, third-party packages like `numpy`, `pybase64`, `pytest`, project helpers such as `tests.utils.RemoteOpenAIServer`.
**CN:** 导入标准库模块（如 `struct`）、第三方包（如 `numpy`、`pybase64`、`pytest`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: DTYPE, MODELS (L19-L31)
```python
DTYPE = "bfloat16"

MODELS: list[tuple[str, list[str]]] = [
    ("intfloat/multilingual-e5-small", []),
    (
        "Snowflake/snowflake-arctic-embed-m-v1.5",
        [
            "--trust_remote_code",
            "--hf_overrides",
            '{"matryoshka_dimensions":[256]}',
        ],
    ),
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `DTYPE`, `MODELS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `DTYPE`、`MODELS`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: model_config (L34-L36)
```python
@pytest.fixture(scope="module", params=MODELS, ids=lambda m: m[0])
def model_config(request):
    return request.param
```
**EN:** This fixture prepares `model_config` for dependent tests. Key inputs are `request`.
**CN:** 这个 fixture 为依赖它的测试准备 `model_config`。 关键输入包括 `request`。

### Fixture / 夹具: model_name (L39-L41)
```python
@pytest.fixture(scope="module")
def model_name(model_config):
    return model_config[0]
```
**EN:** This fixture prepares `model_name` for dependent tests. Key inputs are `model_config`.
**CN:** 这个 fixture 为依赖它的测试准备 `model_name`。 关键输入包括 `model_config`。

### Fixture / 夹具: server (L44-L59)
```python
@pytest.fixture(scope="module")
def server(model_config):
    name, extra_args = model_config
    args = [
        "--runner",
        "pooling",
        "--dtype",
        DTYPE,
        "--enforce-eager",
        "--max-model-len",
        "512",
        "--gpu-memory-utilization",
        "0.02",
    ] + extra_args
    with RemoteOpenAIServer(name, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. Key inputs are `model_config`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 关键输入包括 `model_config`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Helper / 辅助函数: _cohere_embed (L62-L81)
```python
def _cohere_embed(
    server: RemoteOpenAIServer,
    model_name: str,
    texts: list[str] | None = None,
    images: list[str] | None = None,
    input_type: str | None = None,
    embedding_types: list[str] | None = None,
) -> dict:
    body: dict = {"model": model_name}
    if input_type is not None:
        body["input_type"] = input_type
    if texts is not None:
        body["texts"] = texts
    if images is not None:
        body["images"] = images
    if embedding_types is not None:
        body["embedding_types"] = embedding_types
    resp = requests.post(server.url_for("/v2/embed"), json=body)
    resp.raise_for_status()
    return resp.json()
```
**EN:** This helper encapsulates reusable logic in `_cohere_embed`. Key inputs are `server`, `model_name`, `texts`, `images`, `input_type`, `embedding_types`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_cohere_embed` 中。 关键输入包括 `server`、`model_name`、`texts`、`images`、`input_type`、`embedding_types`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _openai_embed (L84-L90)
```python
def _openai_embed(
    server: RemoteOpenAIServer, model_name: str, texts: list[str]
) -> dict:
    body = {"model": model_name, "input": texts, "encoding_format": "float"}
    resp = requests.post(server.url_for("/v1/embeddings"), json=body)
    resp.raise_for_status()
    return resp.json()
```
**EN:** This helper encapsulates reusable logic in `_openai_embed`. Key inputs are `server`, `model_name`, `texts`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_openai_embed` 中。 关键输入包括 `server`、`model_name`、`texts`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _cosine_sim (L93-L95)
```python
def _cosine_sim(a: list[float], b: list[float]) -> float:
    va, vb = np.array(a), np.array(b)
    return float(np.dot(va, vb) / (np.linalg.norm(va) * np.linalg.norm(vb)))
```
**EN:** This helper encapsulates reusable logic in `_cosine_sim`. Key inputs are `a`, `b`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_cosine_sim` 中。 关键输入包括 `a`、`b`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_basic_embed (L103-L109)
```python
def test_basic_embed(server: RemoteOpenAIServer, model_name: str):
    r = _cohere_embed(
        server, model_name, texts=["hello world"], embedding_types=["float"]
    )
    assert "embeddings" in r
    assert len(r["embeddings"]["float"]) == 1
    assert len(r["embeddings"]["float"][0]) > 0
```
**EN:** This test validates `test_basic_embed`. Key inputs are `server`, `model_name`. The main assertion is `'embeddings' in r` and `len(r['embeddings']['float']) == 1`.
**CN:** 这个测试验证 `test_basic_embed`。 关键输入包括 `server`、`model_name`。 核心断言是 `'embeddings' in r` and `len(r['embeddings']['float']) == 1`。

### Test / 测试: test_unsupported_input_type_rejected (L112-L123)
```python
def test_unsupported_input_type_rejected(server: RemoteOpenAIServer, model_name: str):
    """An input_type not defined in the model's prompt config should be
    rejected with a 400 error."""
    body = {
        "model": model_name,
        "input_type": "nonexistent_type",
        "texts": ["hello world"],
        "embedding_types": ["float"],
    }
    resp = requests.post(server.url_for("/v2/embed"), json=body)
    assert resp.status_code == 400
    assert "Unsupported input_type" in resp.json()["error"]["message"]
```
**EN:** This test validates `test_unsupported_input_type_rejected`. Key inputs are `server`, `model_name`. The main assertion is `resp.status_code == 400` and `'Unsupported input_type' in resp.json()['error']['message']`.
**CN:** 这个测试验证 `test_unsupported_input_type_rejected`。 关键输入包括 `server`、`model_name`。 核心断言是 `resp.status_code == 400` and `'Unsupported input_type' in resp.json()['error']['message']`。

### Test / 测试: test_omitted_input_type_accepted (L126-L136)
```python
def test_omitted_input_type_accepted(server: RemoteOpenAIServer, model_name: str):
    """Omitting input_type should always work (no prompt prefix applied)."""
    body = {
        "model": model_name,
        "texts": ["hello world"],
        "embedding_types": ["float"],
    }
    resp = requests.post(server.url_for("/v2/embed"), json=body)
    assert resp.status_code == 200
    data = resp.json()
    assert len(data["embeddings"]["float"]) == 1
```
**EN:** This test validates `test_omitted_input_type_accepted`. Key inputs are `server`, `model_name`. The main assertion is `resp.status_code == 200` and `len(data['embeddings']['float']) == 1`.
**CN:** 这个测试验证 `test_omitted_input_type_accepted`。 关键输入包括 `server`、`model_name`。 核心断言是 `resp.status_code == 200` and `len(data['embeddings']['float']) == 1`。

### Test / 测试: test_v1_v2_parity (L139-L146)
```python
def test_v1_v2_parity(server: RemoteOpenAIServer, model_name: str):
    """v1 (OpenAI) and v2 (Cohere) endpoints should produce the same
    float embeddings for a generic model."""
    texts = ["hello world"]
    v2 = _cohere_embed(server, model_name, texts=texts, embedding_types=["float"])
    v1 = _openai_embed(server, model_name, texts)
    cos = _cosine_sim(v2["embeddings"]["float"][0], v1["data"][0]["embedding"])
    assert cos > 0.9999, f"v1/v2 parity failed, cosine={cos}"
```
**EN:** This test validates `test_v1_v2_parity`. Key inputs are `server`, `model_name`. The main assertion is `cos > 0.9999`.
**CN:** 这个测试验证 `test_v1_v2_parity`。 关键输入包括 `server`、`model_name`。 核心断言是 `cos > 0.9999`。

### Test / 测试: test_embedding_types (L149-L158)
```python
def test_embedding_types(server: RemoteOpenAIServer, model_name: str):
    r = _cohere_embed(
        server,
        model_name,
        texts=["test"],
        embedding_types=["float", "binary", "ubinary"],
    )
    dim = len(r["embeddings"]["float"][0])
    assert len(r["embeddings"]["binary"][0]) == dim // 8
    assert len(r["embeddings"]["ubinary"][0]) == dim // 8
```
**EN:** This test validates `test_embedding_types`. Key inputs are `server`, `model_name`. The main assertion is `len(r['embeddings']['binary'][0]) == dim // 8` and `len(r['embeddings']['ubinary'][0]) == dim // 8`.
**CN:** 这个测试验证 `test_embedding_types`。 关键输入包括 `server`、`model_name`。 核心断言是 `len(r['embeddings']['binary'][0]) == dim // 8` and `len(r['embeddings']['ubinary'][0]) == dim // 8`。

### Test / 测试: test_response_structure (L161-L171)
```python
def test_response_structure(server: RemoteOpenAIServer, model_name: str):
    r = _cohere_embed(server, model_name, texts=["test"], embedding_types=["float"])
    assert "id" in r
    assert "embeddings" in r
    assert "texts" in r
    assert r["texts"] == ["test"]
    assert "meta" in r
    assert r["meta"]["api_version"]["version"] == "2"
    assert "billed_units" in r["meta"]
    assert r["meta"]["billed_units"]["input_tokens"] > 0
    assert r["meta"]["billed_units"]["image_tokens"] == 0
```
**EN:** This test validates `test_response_structure`. Key inputs are `server`, `model_name`. The main assertion is `'id' in r` and `'embeddings' in r`.
**CN:** 这个测试验证 `test_response_structure`。 关键输入包括 `server`、`model_name`。 核心断言是 `'id' in r` and `'embeddings' in r`。

### Test / 测试: test_batch (L174-L180)
```python
def test_batch(server: RemoteOpenAIServer, model_name: str):
    texts = ["apple", "banana", "cherry"]
    r = _cohere_embed(server, model_name, texts=texts, embedding_types=["float"])
    assert len(r["embeddings"]["float"]) == 3
    dim = len(r["embeddings"]["float"][0])
    for emb in r["embeddings"]["float"]:
        assert len(emb) == dim
```
**EN:** This test validates `test_batch`. Key inputs are `server`, `model_name`. The main assertion is `len(r['embeddings']['float']) == 3` and `len(emb) == dim`.
**CN:** 这个测试验证 `test_batch`。 关键输入包括 `server`、`model_name`。 核心断言是 `len(r['embeddings']['float']) == 3` and `len(emb) == dim`。

### Test / 测试: test_l2_normalized (L183-L188)
```python
def test_l2_normalized(server: RemoteOpenAIServer, model_name: str):
    r = _cohere_embed(
        server, model_name, texts=["hello world"], embedding_types=["float"]
    )
    emb = np.array(r["embeddings"]["float"][0])
    assert abs(float(np.linalg.norm(emb)) - 1.0) < 0.01
```
**EN:** This test validates `test_l2_normalized`. Key inputs are `server`, `model_name`. The main assertion is `abs(float(np.linalg.norm(emb)) - 1.0) < 0.01`.
**CN:** 这个测试验证 `test_l2_normalized`。 关键输入包括 `server`、`model_name`。 核心断言是 `abs(float(np.linalg.norm(emb)) - 1.0) < 0.01`。

### Test / 测试: test_semantic_similarity (L191-L201)
```python
def test_semantic_similarity(server: RemoteOpenAIServer, model_name: str):
    r = _cohere_embed(
        server,
        model_name,
        texts=["machine learning", "deep learning", "chocolate cake recipe"],
        embedding_types=["float"],
    )
    embs = r["embeddings"]["float"]
    cos_related = _cosine_sim(embs[0], embs[1])
    cos_unrelated = _cosine_sim(embs[0], embs[2])
    assert cos_related > cos_unrelated
```
**EN:** This test validates `test_semantic_similarity`. Key inputs are `server`, `model_name`. The main assertion is `cos_related > cos_unrelated`.
**CN:** 这个测试验证 `test_semantic_similarity`。 关键输入包括 `server`、`model_name`。 核心断言是 `cos_related > cos_unrelated`。

### Test / 测试: test_missing_input_returns_error (L204-L207)
```python
def test_missing_input_returns_error(server: RemoteOpenAIServer, model_name: str):
    body = {"model": model_name}
    resp = requests.post(server.url_for("/v2/embed"), json=body)
    assert resp.status_code == 400
```
**EN:** This test validates `test_missing_input_returns_error`. Key inputs are `server`, `model_name`. The main assertion is `resp.status_code == 400`.
**CN:** 这个测试验证 `test_missing_input_returns_error`。 关键输入包括 `server`、`model_name`。 核心断言是 `resp.status_code == 400`。

### Test / 测试: test_base64_embedding_type (L210-L220)
```python
def test_base64_embedding_type(server: RemoteOpenAIServer, model_name: str):
    r = _cohere_embed(
        server,
        model_name,
        texts=["test encoding"],
        embedding_types=["float", "base64"],
    )
    float_emb = r["embeddings"]["float"][0]
    b64_str = r["embeddings"]["base64"][0]
    decoded = struct.unpack(f"<{len(float_emb)}f", base64.b64decode(b64_str))
    np.testing.assert_allclose(float_emb, decoded, rtol=1e-5)
```
**EN:** This test validates `test_base64_embedding_type`. Key inputs are `server`, `model_name`.
**CN:** 这个测试验证 `test_base64_embedding_type`。 关键输入包括 `server`、`model_name`。

### Helper / 辅助函数: _cohere_embed_raw (L228-L232)
```python
def _cohere_embed_raw(
    server: RemoteOpenAIServer,
    body: dict,
) -> requests.Response:
    return requests.post(server.url_for("/v2/embed"), json=body)
```
**EN:** This helper encapsulates reusable logic in `_cohere_embed_raw`. Key inputs are `server`, `body`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_cohere_embed_raw` 中。 关键输入包括 `server`、`body`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_truncate_end_succeeds (L235-L247)
```python
def test_truncate_end_succeeds(server: RemoteOpenAIServer, model_name: str):
    """truncate=END should silently truncate long input."""
    long_text = " ".join(["word"] * 2000)
    body = {
        "model": model_name,
        "texts": [long_text],
        "embedding_types": ["float"],
        "truncate": "END",
    }
    resp = _cohere_embed_raw(server, body)
    assert resp.status_code == 200
    data = resp.json()
    assert len(data["embeddings"]["float"]) == 1
```
**EN:** This test validates `test_truncate_end_succeeds`. Key inputs are `server`, `model_name`. The main assertion is `resp.status_code == 200` and `len(data['embeddings']['float']) == 1`.
**CN:** 这个测试验证 `test_truncate_end_succeeds`。 关键输入包括 `server`、`model_name`。 核心断言是 `resp.status_code == 200` and `len(data['embeddings']['float']) == 1`。

### Test / 测试: test_truncate_start_succeeds (L250-L262)
```python
def test_truncate_start_succeeds(server: RemoteOpenAIServer, model_name: str):
    """truncate=START should silently truncate long input from the start."""
    long_text = " ".join(["word"] * 2000)
    body = {
        "model": model_name,
        "texts": [long_text],
        "embedding_types": ["float"],
        "truncate": "START",
    }
    resp = _cohere_embed_raw(server, body)
    assert resp.status_code == 200
    data = resp.json()
    assert len(data["embeddings"]["float"]) == 1
```
**EN:** This test validates `test_truncate_start_succeeds`. Key inputs are `server`, `model_name`. The main assertion is `resp.status_code == 200` and `len(data['embeddings']['float']) == 1`.
**CN:** 这个测试验证 `test_truncate_start_succeeds`。 关键输入包括 `server`、`model_name`。 核心断言是 `resp.status_code == 200` and `len(data['embeddings']['float']) == 1`。

### Test / 测试: test_truncate_none_rejects_long_input (L265-L275)
```python
def test_truncate_none_rejects_long_input(server: RemoteOpenAIServer, model_name: str):
    """truncate=NONE should error when input exceeds model context."""
    long_text = " ".join(["word"] * 2000)
    body = {
        "model": model_name,
        "texts": [long_text],
        "embedding_types": ["float"],
        "truncate": "NONE",
    }
    resp = _cohere_embed_raw(server, body)
    assert resp.status_code == 400
```
**EN:** This test validates `test_truncate_none_rejects_long_input`. Key inputs are `server`, `model_name`. The main assertion is `resp.status_code == 400`.
**CN:** 这个测试验证 `test_truncate_none_rejects_long_input`。 关键输入包括 `server`、`model_name`。 核心断言是 `resp.status_code == 400`。

### Test / 测试: test_truncate_start_vs_end_differ (L278-L310)
```python
def test_truncate_start_vs_end_differ(server: RemoteOpenAIServer, model_name: str):
    """START and END truncation should produce different embeddings
    when the input is long enough to actually be truncated.

    We construct input with distinct tokens at the start vs end
    so that keeping different halves produces different embeddings.
    """
    start_words = " ".join([f"alpha{i}" for i in range(300)])
    end_words = " ".join([f"omega{i}" for i in range(300)])
    long_text = start_words + " " + end_words

    body_end = {
        "model": model_name,
        "texts": [long_text],
        "embedding_types": ["float"],
        "truncate": "END",
    }
    body_start = {
# ... 7 lines omitted for brevity ...

    emb_end = r_end["embeddings"]["float"][0]
    emb_start = r_start["embeddings"]["float"][0]
    cos = _cosine_sim(emb_end, emb_start)
    assert cos < 0.99, (
        f"START and END truncation should produce different embeddings "
        f"for long input, but cosine similarity was {cos}"
    )
```
**EN:** This test validates `test_truncate_start_vs_end_differ`. Key inputs are `server`, `model_name`. The main assertion is `cos < 0.99`.
**CN:** 这个测试验证 `test_truncate_start_vs_end_differ`。 关键输入包括 `server`、`model_name`。 核心断言是 `cos < 0.99`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `struct`
- **Third-party / 第三方**: `numpy`, `pybase64`, `pytest`, `requests`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`
