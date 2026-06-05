# test_cohere_online_vision.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/embed/test_cohere_online_vision.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and vision or image inputs. The file defines 5 test(s), 1 fixture(s), and 3 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与视觉或图像输入。它定义了 5 个测试、1 个 fixture，以及 3 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L9-L17)
```python
import struct
import zlib

import numpy as np
import pybase64 as base64
import pytest
import requests

from tests.utils import RemoteOpenAIServer
```
**EN:** Imports standard-library modules such as `struct`, `zlib`, third-party packages like `numpy`, `pybase64`, `pytest`, project helpers such as `tests.utils.RemoteOpenAIServer`.
**CN:** 导入标准库模块（如 `struct`、`zlib`）、第三方包（如 `numpy`、`pybase64`、`pytest`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: MODEL_NAME, DTYPE (L19-L20)
```python
MODEL_NAME = "google/siglip-so400m-patch14-384"
DTYPE = "bfloat16"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `DTYPE`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`DTYPE`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L23-L37)
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
        "64",
        "--gpu-memory-utilization",
        "0.3",
    ]
    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Helper / 辅助函数: _make_tiny_png (L40-L61)
```python
def _make_tiny_png(r: int, g: int, b: int, w: int = 2, h: int = 2) -> str:
    raw = b""
    for _ in range(h):
        raw += b"\x00" + bytes([r, g, b]) * w
    compressed = zlib.compress(raw)

    def chunk(ctype: bytes, cdata: bytes) -> bytes:
        c = ctype + cdata
        return (
            struct.pack(">I", len(cdata))
            + c
            + struct.pack(">I", zlib.crc32(c) & 0xFFFFFFFF)
        )

    ihdr = struct.pack(">IIBBBBB", w, h, 8, 2, 0, 0, 0)
    png = (
        b"\x89PNG\r\n\x1a\n"
        + chunk(b"IHDR", ihdr)
        + chunk(b"IDAT", compressed)
        + chunk(b"IEND", b"")
    )
    return "data:image/png;base64," + base64.b64encode(png).decode()
```
**EN:** This helper encapsulates reusable logic in `_make_tiny_png`. Key inputs are `r`, `g`, `b`, `w`, `h`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_make_tiny_png` 中。 关键输入包括 `r`、`g`、`b`、`w`、`h`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _cohere_embed (L64-L79)
```python
def _cohere_embed(
    server: RemoteOpenAIServer,
    texts: list[str] | None = None,
    images: list[str] | None = None,
    embedding_types: list[str] | None = None,
) -> dict:
    body: dict = {"model": MODEL_NAME}
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
**EN:** This helper encapsulates reusable logic in `_cohere_embed`. Key inputs are `server`, `texts`, `images`, `embedding_types`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_cohere_embed` 中。 关键输入包括 `server`、`texts`、`images`、`embedding_types`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_image_embed (L82-L93)
```python
def test_image_embed(server: RemoteOpenAIServer):
    img_uri = _make_tiny_png(255, 0, 0)
    r = _cohere_embed(
        server,
        images=[img_uri],
        embedding_types=["float"],
    )
    assert "embeddings" in r
    assert len(r["embeddings"]["float"]) == 1
    assert len(r["embeddings"]["float"][0]) > 0
    assert r["meta"]["billed_units"]["image_tokens"] > 0
    assert r["meta"]["billed_units"]["input_tokens"] == 0
```
**EN:** This test validates `test_image_embed`. Key inputs are `server`. The main assertion is `'embeddings' in r` and `len(r['embeddings']['float']) == 1`.
**CN:** 这个测试验证 `test_image_embed`。 关键输入包括 `server`。 核心断言是 `'embeddings' in r` and `len(r['embeddings']['float']) == 1`。

### Test / 测试: test_image_batch (L96-L104)
```python
def test_image_batch(server: RemoteOpenAIServer):
    red = _make_tiny_png(255, 0, 0)
    blue = _make_tiny_png(0, 0, 255)
    r = _cohere_embed(
        server,
        images=[red, blue],
        embedding_types=["float"],
    )
    assert len(r["embeddings"]["float"]) == 2
```
**EN:** This test validates `test_image_batch`. Key inputs are `server`. The main assertion is `len(r['embeddings']['float']) == 2`.
**CN:** 这个测试验证 `test_image_batch`。 关键输入包括 `server`。 核心断言是 `len(r['embeddings']['float']) == 2`。

### Test / 测试: test_image_l2_normalized (L107-L115)
```python
def test_image_l2_normalized(server: RemoteOpenAIServer):
    img_uri = _make_tiny_png(0, 255, 0)
    r = _cohere_embed(
        server,
        images=[img_uri],
        embedding_types=["float"],
    )
    emb = np.array(r["embeddings"]["float"][0])
    assert abs(float(np.linalg.norm(emb)) - 1.0) < 0.01
```
**EN:** This test validates `test_image_l2_normalized`. Key inputs are `server`. The main assertion is `abs(float(np.linalg.norm(emb)) - 1.0) < 0.01`.
**CN:** 这个测试验证 `test_image_l2_normalized`。 关键输入包括 `server`。 核心断言是 `abs(float(np.linalg.norm(emb)) - 1.0) < 0.01`。

### Test / 测试: test_image_embedding_types (L118-L127)
```python
def test_image_embedding_types(server: RemoteOpenAIServer):
    img_uri = _make_tiny_png(128, 128, 128)
    r = _cohere_embed(
        server,
        images=[img_uri],
        embedding_types=["float", "binary", "ubinary"],
    )
    dim = len(r["embeddings"]["float"][0])
    assert len(r["embeddings"]["binary"][0]) == dim // 8
    assert len(r["embeddings"]["ubinary"][0]) == dim // 8
```
**EN:** This test validates `test_image_embedding_types`. Key inputs are `server`. The main assertion is `len(r['embeddings']['binary'][0]) == dim // 8` and `len(r['embeddings']['ubinary'][0]) == dim // 8`.
**CN:** 这个测试验证 `test_image_embedding_types`。 关键输入包括 `server`。 核心断言是 `len(r['embeddings']['binary'][0]) == dim // 8` and `len(r['embeddings']['ubinary'][0]) == dim // 8`。

### Test / 测试: test_text_embed_on_multimodal (L130-L135)
```python
def test_text_embed_on_multimodal(server: RemoteOpenAIServer):
    """SigLIP also supports text-only embedding via /v2/embed."""
    r = _cohere_embed(server, texts=["hello world"], embedding_types=["float"])
    assert "embeddings" in r
    assert len(r["embeddings"]["float"]) == 1
    assert len(r["embeddings"]["float"][0]) > 0
```
**EN:** This test validates `test_text_embed_on_multimodal`. Key inputs are `server`. The main assertion is `'embeddings' in r` and `len(r['embeddings']['float']) == 1`.
**CN:** 这个测试验证 `test_text_embed_on_multimodal`。 关键输入包括 `server`。 核心断言是 `'embeddings' in r` and `len(r['embeddings']['float']) == 1`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `struct`, `zlib`
- **Third-party / 第三方**: `numpy`, `pybase64`, `pytest`, `requests`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`
