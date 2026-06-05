# test_chat_completion_with_mixed_audio_embeds.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/chat_completion/test_chat_completion_with_mixed_audio_embeds.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers chat completion behavior and text completion behavior. The file defines 2 test(s), 7 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖聊天补全行为与文本补全行为。它定义了 2 个测试、7 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L7-L19)
```python
import json

import openai
import pytest
import pytest_asyncio
import safetensors
import torch
import torch.nn as nn
from huggingface_hub import hf_hub_download
from transformers import AutoConfig, AutoTokenizer

from tests.utils import RemoteOpenAIServer
from vllm.utils.serial_utils import tensor2base64
```
**EN:** Imports standard-library modules such as `json`, third-party packages like `huggingface_hub.hf_hub_download`, `openai`, `pytest`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.utils.serial_utils.tensor2base64`.
**CN:** 导入标准库模块（如 `json`）、第三方包（如 `huggingface_hub.hf_hub_download`、`openai`、`pytest`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.utils.serial_utils.tensor2base64`）。

### Module setup / 模块级配置: QWEN2AUDIO_MODEL, QWEN2AUDIO_DTYPE (L21-L26)
```python
QWEN2AUDIO_MODEL = "Qwen/Qwen2-Audio-7B-Instruct"

# Use the model's native dtype to avoid an implicit cast inside
# `safe_load_prompt_embeds` (mismatched floating-point dtypes are cast to the
# model's dtype automatically, matching here just skips the conversion).
QWEN2AUDIO_DTYPE = torch.bfloat16
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `QWEN2AUDIO_MODEL`, `QWEN2AUDIO_DTYPE`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `QWEN2AUDIO_MODEL`、`QWEN2AUDIO_DTYPE`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: qwen2audio_server_args (L29-L46)
```python
@pytest.fixture(scope="module")
def qwen2audio_server_args() -> list[str]:
    return [
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "2048",
        "--max-num-seqs",
        "4",
        "--enforce-eager",
        "--trust-remote-code",
        "--gpu-memory-utilization",
        "0.85",
        "--limit-mm-per-prompt",
        json.dumps({"audio": 1}),
        "--enable-prompt-embeds",
        "--enable-mm-embeds",
    ]
```
**EN:** This fixture prepares `qwen2audio_server_args` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `qwen2audio_server_args`。

### Fixture / 夹具: qwen2audio_server (L49-L56)
```python
@pytest.fixture(scope="module")
def qwen2audio_server(qwen2audio_server_args):
    with RemoteOpenAIServer(
        QWEN2AUDIO_MODEL,
        qwen2audio_server_args,
        max_wait_seconds=600,
    ) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `qwen2audio_server` for dependent tests. Key inputs are `qwen2audio_server_args`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `qwen2audio_server`。 关键输入包括 `qwen2audio_server_args`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: qwen2audio_client (L59-L62)
```python
@pytest_asyncio.fixture
async def qwen2audio_client(qwen2audio_server):
    async with qwen2audio_server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `qwen2audio_client` for dependent tests. Key inputs are `qwen2audio_server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `qwen2audio_client`。 关键输入包括 `qwen2audio_server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: qwen2audio_hidden_size (L65-L68)
```python
@pytest.fixture(scope="module")
def qwen2audio_hidden_size() -> int:
    config = AutoConfig.from_pretrained(QWEN2AUDIO_MODEL, trust_remote_code=True)
    return config.text_config.hidden_size
```
**EN:** This fixture prepares `qwen2audio_hidden_size` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `qwen2audio_hidden_size`。

### Fixture / 夹具: qwen2audio_prompt_embeds_b64 (L71-L74)
```python
@pytest.fixture(scope="module")
def qwen2audio_prompt_embeds_b64(qwen2audio_hidden_size: int) -> str:
    tensor = torch.randn(4, qwen2audio_hidden_size, dtype=QWEN2AUDIO_DTYPE)
    return tensor2base64(tensor)
```
**EN:** This fixture prepares `qwen2audio_prompt_embeds_b64` for dependent tests. Key inputs are `qwen2audio_hidden_size`.
**CN:** 这个 fixture 为依赖它的测试准备 `qwen2audio_prompt_embeds_b64`。 关键输入包括 `qwen2audio_hidden_size`。

### Fixture / 夹具: qwen2audio_audio_embeds_b64 (L77-L82)
```python
@pytest.fixture(scope="module")
def qwen2audio_audio_embeds_b64(qwen2audio_hidden_size: int) -> str:
    # Shape matches the `audio_embeds` unit-test fixture.
    torch.manual_seed(0)
    tensor = torch.randn(1, 128, qwen2audio_hidden_size, dtype=QWEN2AUDIO_DTYPE)
    return tensor2base64(tensor)
```
**EN:** This fixture prepares `qwen2audio_audio_embeds_b64` for dependent tests. Key inputs are `qwen2audio_hidden_size`.
**CN:** 这个 fixture 为依赖它的测试准备 `qwen2audio_audio_embeds_b64`。 关键输入包括 `qwen2audio_hidden_size`。

### Test / 测试: test_prompt_embeds_plus_audio_embeds (L85-L114)
```python
@pytest.mark.asyncio
async def test_prompt_embeds_plus_audio_embeds(
    qwen2audio_client: openai.AsyncOpenAI,
    qwen2audio_prompt_embeds_b64: str,
    qwen2audio_audio_embeds_b64: str,
):
    """Single user message carrying both prompt_embeds and audio_embeds parts."""
    chat = await qwen2audio_client.chat.completions.create(
        model=QWEN2AUDIO_MODEL,
        max_tokens=5,
        temperature=0.0,
        messages=[
            {
                "role": "user",
                "content": [
                    {
                        "type": "prompt_embeds",
                        "data": qwen2audio_prompt_embeds_b64,
                    },
                    {
                        "type": "audio_embeds",
                        "audio_embeds": qwen2audio_audio_embeds_b64,
                    },
                    {"type": "text", "text": "Continue."},
                ],
            }
        ],
    )
    assert chat.choices[0].message.content is not None
    assert len(chat.choices[0].message.content) > 0
```
**EN:** This async test validates `test_prompt_embeds_plus_audio_embeds`. Relevant pytest markers include `asyncio`. Key inputs are `qwen2audio_client`, `qwen2audio_prompt_embeds_b64`, `qwen2audio_audio_embeds_b64`. It drives client-facing request creation through the API surface under test. The main assertion is `chat.choices[0].message.content is not None` and `len(chat.choices[0].message.content) > 0`.
**CN:** 这个异步测试验证 `test_prompt_embeds_plus_audio_embeds`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `qwen2audio_client`、`qwen2audio_prompt_embeds_b64`、`qwen2audio_audio_embeds_b64`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `chat.choices[0].message.content is not None` and `len(chat.choices[0].message.content) > 0`。

### Fixture / 夹具: qwen2audio_aligned_content_and_embeds_b64 (L117-L139)
```python
@pytest.fixture(scope="module")
def qwen2audio_aligned_content_and_embeds_b64() -> tuple[str, str]:
    """Return `(content, base64_embeds)` where the embeddings are the model's
    embedding of `content` tokenized WITHOUT special tokens.

    Loads only the `embed_tokens` shard from disk on CPU (~1.1 GB of host
    RAM) instead of the full 7B model on GPU.
    """
    content = "Describe this audio."
    tokenizer = AutoTokenizer.from_pretrained(QWEN2AUDIO_MODEL, trust_remote_code=True)

    index_path = hf_hub_download(QWEN2AUDIO_MODEL, "model.safetensors.index.json")
    with open(index_path) as f:
        weight_map = json.load(f)["weight_map"]
    embed_key = next(k for k in weight_map if k.endswith("embed_tokens.weight"))
    shard_path = hf_hub_download(QWEN2AUDIO_MODEL, weight_map[embed_key])
    with safetensors.safe_open(shard_path, framework="pt", device="cpu") as f:
        embed_weight = f.get_tensor(embed_key)
    embed_layer = nn.Embedding.from_pretrained(embed_weight.to(QWEN2AUDIO_DTYPE))

    ids = tokenizer(content, add_special_tokens=False, return_tensors="pt").input_ids
    embeds = embed_layer(ids).squeeze(0)
    return content, tensor2base64(embeds)
```
**EN:** This fixture prepares `qwen2audio_aligned_content_and_embeds_b64` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `qwen2audio_aligned_content_and_embeds_b64`。

### Test / 测试: test_text_content_and_prompt_embeds_match_with_audio_embeds (L142-L190)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "audio_first",
    [True, False],
    ids=["audio_embeds-then-text", "text-then-audio_embeds"],
)
async def test_text_content_and_prompt_embeds_match_with_audio_embeds(
    qwen2audio_client: openai.AsyncOpenAI,
    qwen2audio_audio_embeds_b64: str,
    qwen2audio_aligned_content_and_embeds_b64: tuple[str, str],
    audio_first: bool,
):
    """Same content as text vs `prompt_embeds` should yield identical Chat
    Completions output when mixed with `audio_embeds` in the same message.
    """
    content, encoded_text_embeds = qwen2audio_aligned_content_and_embeds_b64

    audio_part = {
# ... 23 lines omitted for brevity ...
        messages=[{"role": "user", "content": embeds_content}],
    )

    text_out = text_resp.choices[0].message.content
    embeds_out = embeds_resp.choices[0].message.content
    assert text_out is not None and len(text_out) > 0
    assert embeds_out is not None and len(embeds_out) > 0
    assert text_out == embeds_out
```
**EN:** This async test validates `test_text_content_and_prompt_embeds_match_with_audio_embeds`. It uses parameterization to cover `audio_embeds-then-text`, `text-then-audio_embeds`. Relevant pytest markers include `asyncio`. Key inputs are `qwen2audio_client`, `qwen2audio_audio_embeds_b64`, `qwen2audio_aligned_content_and_embeds_b64`, `audio_first`. It drives client-facing request creation through the API surface under test. The main assertion is `text_out is not None and len(text_out) > 0` and `embeds_out is not None and len(embeds_out) > 0`.
**CN:** 这个异步测试验证 `test_text_content_and_prompt_embeds_match_with_audio_embeds`。 它通过参数化覆盖 `audio_embeds-then-text`、`text-then-audio_embeds` 等场景。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `qwen2audio_client`、`qwen2audio_audio_embeds_b64`、`qwen2audio_aligned_content_and_embeds_b64`、`audio_first`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `text_out is not None and len(text_out) > 0` and `embeds_out is not None and len(embeds_out) > 0`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`
- **Third-party / 第三方**: `huggingface_hub.hf_hub_download`, `openai`, `pytest`, `pytest_asyncio`, `safetensors`, `torch`, `torch.nn`, `transformers.AutoConfig`, `transformers.AutoTokenizer`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.utils.serial_utils.tensor2base64`
