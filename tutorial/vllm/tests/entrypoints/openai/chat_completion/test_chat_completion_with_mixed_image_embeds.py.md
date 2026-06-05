# test_chat_completion_with_mixed_image_embeds.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/chat_completion/test_chat_completion_with_mixed_image_embeds.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers chat completion behavior and text completion behavior. The file defines 2 test(s), 6 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖聊天补全行为与文本补全行为。它定义了 2 个测试、6 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L8-L22)
```python
import json

import openai
import pytest
import pytest_asyncio
import safetensors
import torch
import torch.nn as nn
from huggingface_hub import hf_hub_download
from transformers import AutoTokenizer

from tests.utils import RemoteOpenAIServer
from vllm.assets.image import ImageAsset
from vllm.multimodal.utils import encode_image_url
from vllm.utils.serial_utils import tensor2base64
```
**EN:** Imports standard-library modules such as `json`, third-party packages like `huggingface_hub.hf_hub_download`, `openai`, `pytest`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.assets.image.ImageAsset`, `vllm.multimodal.utils.encode_image_url`.
**CN:** 导入标准库模块（如 `json`）、第三方包（如 `huggingface_hub.hf_hub_download`、`openai`、`pytest`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.assets.image.ImageAsset`、`vllm.multimodal.utils.encode_image_url`）。

### Module setup / 模块级配置: MODEL_NAME, MODEL_DTYPE (L24-L29)
```python
MODEL_NAME = "Qwen/Qwen2-VL-2B-Instruct"

# Use the model's native dtype to skip the implicit cast inside
# `safe_load_prompt_embeds` (mismatched floating-point dtypes are cast to the
# model's dtype automatically).
MODEL_DTYPE = torch.bfloat16
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `MODEL_DTYPE`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`MODEL_DTYPE`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server_args (L32-L48)
```python
@pytest.fixture(scope="module")
def server_args() -> list[str]:
    return [
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "2048",
        "--max-num-seqs",
        "4",
        "--enforce-eager",
        "--gpu-memory-utilization",
        "0.4",
        "--limit-mm-per-prompt",
        json.dumps({"image": 1}),
        "--enable-prompt-embeds",
        "--enable-mm-embeds",
    ]
```
**EN:** This fixture prepares `server_args` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `server_args`。

### Fixture / 夹具: server (L51-L58)
```python
@pytest.fixture(scope="module")
def server(server_args):
    with RemoteOpenAIServer(
        MODEL_NAME,
        server_args,
        max_wait_seconds=600,
    ) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. Key inputs are `server_args`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 关键输入包括 `server_args`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L61-L64)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: image_url (L67-L72)
```python
@pytest.fixture(scope="module")
def image_url() -> str:
    """Stable real image as a data URL, kept identical across both the
    text and prompt_embeds requests so any output difference must come from
    how the text content is delivered."""
    return encode_image_url(ImageAsset("stop_sign").pil_image)
```
**EN:** This fixture prepares `image_url` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `image_url`。

### Fixture / 夹具: aligned_content_and_embeds_b64 (L75-L98)
```python
@pytest.fixture(scope="module")
def aligned_content_and_embeds_b64() -> tuple[str, str]:
    """`(content, base64_embeds)` where the embeddings are the model's
    embedding of `content` tokenized WITHOUT special tokens.

    Loads only the `embed_tokens` shard from disk on CPU instead of the full
    model on GPU, so the fixture has zero VRAM footprint and won't contend
    with the running vLLM server.
    """
    content = "Describe this image."
    tokenizer = AutoTokenizer.from_pretrained(MODEL_NAME, trust_remote_code=True)

    index_path = hf_hub_download(MODEL_NAME, "model.safetensors.index.json")
    with open(index_path) as f:
        weight_map = json.load(f)["weight_map"]
    embed_key = next(k for k in weight_map if k.endswith("embed_tokens.weight"))
    shard_path = hf_hub_download(MODEL_NAME, weight_map[embed_key])
    with safetensors.safe_open(shard_path, framework="pt", device="cpu") as f:
        embed_weight = f.get_tensor(embed_key)
    embed_layer = nn.Embedding.from_pretrained(embed_weight.to(MODEL_DTYPE))

    ids = tokenizer(content, add_special_tokens=False, return_tensors="pt").input_ids
    embeds = embed_layer(ids).squeeze(0)
    return content, tensor2base64(embeds)
```
**EN:** This fixture prepares `aligned_content_and_embeds_b64` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `aligned_content_and_embeds_b64`。

### Test / 测试: test_text_content_and_prompt_embeds_match_with_image_url (L101-L147)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "image_first",
    [True, False],
    ids=["image_url-then-text", "text-then-image_url"],
)
async def test_text_content_and_prompt_embeds_match_with_image_url(
    client: openai.AsyncOpenAI,
    image_url: str,
    aligned_content_and_embeds_b64: tuple[str, str],
    image_first: bool,
):
    """Same content as text vs `prompt_embeds` should yield identical Chat
    Completions output when mixed with an `image_url` part in the same
    message under greedy decoding.
    """
    content, encoded_text_embeds = aligned_content_and_embeds_b64

# ... 21 lines omitted for brevity ...
        messages=[{"role": "user", "content": embeds_content}],
    )

    text_out = text_resp.choices[0].message.content
    embeds_out = embeds_resp.choices[0].message.content
    assert text_out is not None and len(text_out) > 0
    assert embeds_out is not None and len(embeds_out) > 0
    assert text_out == embeds_out
```
**EN:** This async test validates `test_text_content_and_prompt_embeds_match_with_image_url`. It uses parameterization to cover `image_url-then-text`, `text-then-image_url`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `image_url`, `aligned_content_and_embeds_b64`, `image_first`. It drives client-facing request creation through the API surface under test. The main assertion is `text_out is not None and len(text_out) > 0` and `embeds_out is not None and len(embeds_out) > 0`.
**CN:** 这个异步测试验证 `test_text_content_and_prompt_embeds_match_with_image_url`。 它通过参数化覆盖 `image_url-then-text`、`text-then-image_url` 等场景。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`image_url`、`aligned_content_and_embeds_b64`、`image_first`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `text_out is not None and len(text_out) > 0` and `embeds_out is not None and len(embeds_out) > 0`。

### Fixture / 夹具: image_embeds_b64 (L150-L163)
```python
@pytest.fixture(scope="module")
def image_embeds_b64() -> dict[str, str]:
    """Synthetic but stable `image_embeds` for Qwen2-VL."""
    grid = (1, 4, 4)
    spatial_merge_size = 2
    num_patches = (grid[1] // spatial_merge_size) * (grid[2] // spatial_merge_size)
    text_hidden_size = 1536  # Qwen2-VL-2B
    torch.manual_seed(0)
    return {
        "image_embeds": tensor2base64(
            torch.randn(num_patches, text_hidden_size, dtype=MODEL_DTYPE)
        ),
        "image_grid_thw": tensor2base64(torch.tensor(grid)),
    }
```
**EN:** This fixture prepares `image_embeds_b64` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `image_embeds_b64`。

### Test / 测试: test_text_content_and_prompt_embeds_match_with_image_embeds (L166-L212)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "image_first",
    [True, False],
    ids=["image_embeds-then-text", "text-then-image_embeds"],
)
async def test_text_content_and_prompt_embeds_match_with_image_embeds(
    client: openai.AsyncOpenAI,
    image_embeds_b64: dict[str, str],
    aligned_content_and_embeds_b64: tuple[str, str],
    image_first: bool,
):
    """Same content as text vs `prompt_embeds` should yield identical Chat
    Completions output when mixed with a precomputed `image_embeds` part in
    the same message under greedy decoding.
    """
    content, encoded_text_embeds = aligned_content_and_embeds_b64

# ... 21 lines omitted for brevity ...
        messages=[{"role": "user", "content": embeds_content}],
    )

    text_out = text_resp.choices[0].message.content
    embeds_out = embeds_resp.choices[0].message.content
    assert text_out is not None and len(text_out) > 0
    assert embeds_out is not None and len(embeds_out) > 0
    assert text_out == embeds_out
```
**EN:** This async test validates `test_text_content_and_prompt_embeds_match_with_image_embeds`. It uses parameterization to cover `image_embeds-then-text`, `text-then-image_embeds`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `image_embeds_b64`, `aligned_content_and_embeds_b64`, `image_first`. It drives client-facing request creation through the API surface under test. The main assertion is `text_out is not None and len(text_out) > 0` and `embeds_out is not None and len(embeds_out) > 0`.
**CN:** 这个异步测试验证 `test_text_content_and_prompt_embeds_match_with_image_embeds`。 它通过参数化覆盖 `image_embeds-then-text`、`text-then-image_embeds` 等场景。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`image_embeds_b64`、`aligned_content_and_embeds_b64`、`image_first`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `text_out is not None and len(text_out) > 0` and `embeds_out is not None and len(embeds_out) > 0`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`
- **Third-party / 第三方**: `huggingface_hub.hf_hub_download`, `openai`, `pytest`, `pytest_asyncio`, `safetensors`, `torch`, `torch.nn`, `transformers.AutoTokenizer`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.assets.image.ImageAsset`, `vllm.multimodal.utils.encode_image_url`, `vllm.utils.serial_utils.tensor2base64`
