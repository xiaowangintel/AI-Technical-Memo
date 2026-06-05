# test_cohere_openai_parity.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/embed/test_cohere_openai_parity.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and OpenAI-compatible serving. The file defines 3 test(s), 1 fixture(s), and 3 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与OpenAI 兼容服务。它定义了 3 个测试、1 个 fixture，以及 3 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L9-L13)
```python
import numpy as np
import pytest
import requests

from tests.utils import ROCM_EXTRA_ARGS, RemoteOpenAIServer
```
**EN:** Imports third-party packages like `numpy`, `pytest`, `requests`, project helpers such as `tests.utils.ROCM_EXTRA_ARGS`, `tests.utils.RemoteOpenAIServer`.
**CN:** 导入第三方包（如 `numpy`、`pytest`、`requests`）、项目内辅助模块（如 `tests.utils.ROCM_EXTRA_ARGS`、`tests.utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: MODEL_NAME, DTYPE (L15-L16)
```python
MODEL_NAME = "BAAI/bge-base-en-v1.5"
DTYPE = "bfloat16"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `DTYPE`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`DTYPE`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L19-L33)
```python
@pytest.fixture(scope="module")
def server():
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
    ] + ROCM_EXTRA_ARGS
    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Helper / 辅助函数: _cohere_embed (L36-L47)
```python
def _cohere_embed(
    server: RemoteOpenAIServer,
    texts: list[str],
) -> list[list[float]]:
    body = {
        "model": MODEL_NAME,
        "texts": texts,
        "embedding_types": ["float"],
    }
    resp = requests.post(server.url_for("/v2/embed"), json=body)
    resp.raise_for_status()
    return resp.json()["embeddings"]["float"]
```
**EN:** This helper encapsulates reusable logic in `_cohere_embed`. Key inputs are `server`, `texts`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_cohere_embed` 中。 关键输入包括 `server`、`texts`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _openai_embed (L50-L57)
```python
def _openai_embed(
    server: RemoteOpenAIServer,
    texts: list[str],
) -> list[list[float]]:
    body = {"model": MODEL_NAME, "input": texts, "encoding_format": "float"}
    resp = requests.post(server.url_for("/v1/embeddings"), json=body)
    resp.raise_for_status()
    return [item["embedding"] for item in resp.json()["data"]]
```
**EN:** This helper encapsulates reusable logic in `_openai_embed`. Key inputs are `server`, `texts`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_openai_embed` 中。 关键输入包括 `server`、`texts`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _cosine_sim (L60-L62)
```python
def _cosine_sim(a: list[float], b: list[float]) -> float:
    va, vb = np.array(a), np.array(b)
    return float(np.dot(va, vb) / (np.linalg.norm(va) * np.linalg.norm(vb)))
```
**EN:** This helper encapsulates reusable logic in `_cosine_sim`. Key inputs are `a`, `b`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_cosine_sim` 中。 关键输入包括 `a`、`b`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_single_text_parity (L65-L74)
```python
def test_single_text_parity(server: RemoteOpenAIServer):
    """A single text should produce equivalent embeddings via both APIs."""
    texts = ["the quick brown fox jumps over the lazy dog"]
    v2 = _cohere_embed(server, texts)
    v1 = _openai_embed(server, texts)
    # Full-suite BF16 runs can introduce tiny numerical drift even when both
    # endpoints are functionally equivalent, so compare semantic equivalence
    # instead of exact elementwise equality.
    cos = _cosine_sim(v2[0], v1[0])
    assert cos > 0.9999, f"single-text parity failed, cosine={cos}"
```
**EN:** This test validates `test_single_text_parity`. Key inputs are `server`. The main assertion is `cos > 0.9999`.
**CN:** 这个测试验证 `test_single_text_parity`。 关键输入包括 `server`。 核心断言是 `cos > 0.9999`。

### Test / 测试: test_batch_parity (L77-L99)
```python
def test_batch_parity(server: RemoteOpenAIServer):
    """A batch of texts should produce equivalent embeddings via both APIs,
    in the same order."""
    texts = [
        "machine learning",
        "deep learning",
        "natural language processing",
    ]
    v2 = _cohere_embed(server, texts)
    v1 = _openai_embed(server, texts)
    assert len(v2) == len(v1) == 3

    similarities = np.array(
        [[_cosine_sim(v2_emb, v1_emb) for v1_emb in v1] for v2_emb in v2]
    )
    for i in range(3):
        assert int(np.argmax(similarities[i])) == i, (
            f"batch parity order mismatch at index {i}: "
            f"similarities={similarities[i].tolist()}"
        )
        assert similarities[i, i] > 0.9999, (
            f"batch parity failed at index {i}, cosine={similarities[i, i]}"
        )
```
**EN:** This test validates `test_batch_parity`. Key inputs are `server`. The main assertion is `len(v2) == len(v1) == 3` and `int(np.argmax(similarities[i])) == i`.
**CN:** 这个测试验证 `test_batch_parity`。 关键输入包括 `server`。 核心断言是 `len(v2) == len(v1) == 3` and `int(np.argmax(similarities[i])) == i`。

### Test / 测试: test_token_count_parity (L102-L121)
```python
def test_token_count_parity(server: RemoteOpenAIServer):
    """Both APIs should report the same prompt token count."""
    texts = ["hello world"]
    v2_resp = requests.post(
        server.url_for("/v2/embed"),
        json={
            "model": MODEL_NAME,
            "texts": texts,
            "embedding_types": ["float"],
        },
    )
    v1_resp = requests.post(
        server.url_for("/v1/embeddings"),
        json={"model": MODEL_NAME, "input": texts, "encoding_format": "float"},
    )
    v2_resp.raise_for_status()
    v1_resp.raise_for_status()
    v2_tokens = v2_resp.json()["meta"]["billed_units"]["input_tokens"]
    v1_tokens = v1_resp.json()["usage"]["prompt_tokens"]
    assert v2_tokens == v1_tokens
```
**EN:** This test validates `test_token_count_parity`. Key inputs are `server`. The main assertion is `v2_tokens == v1_tokens`.
**CN:** 这个测试验证 `test_token_count_parity`。 关键输入包括 `server`。 核心断言是 `v2_tokens == v1_tokens`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `numpy`, `pytest`, `requests`
- **Project / 项目内**: `tests.utils.ROCM_EXTRA_ARGS`, `tests.utils.RemoteOpenAIServer`
