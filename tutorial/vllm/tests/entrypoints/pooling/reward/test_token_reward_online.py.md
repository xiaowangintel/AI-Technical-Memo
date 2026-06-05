# test_token_reward_online.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/reward/test_token_reward_online.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior. The file defines 11 test(s), 1 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为。它定义了 11 个测试、1 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L21)
```python
import json

import numpy as np
import pybase64 as base64
import pytest
import requests
import torch

from tests.models.utils import check_embeddings_close
from tests.utils import RemoteOpenAIServer
from vllm.entrypoints.pooling.pooling.protocol import PoolingResponse
from vllm.entrypoints.pooling.utils import (
    MetadataItem,
    build_metadata_items,
    decode_pooling_output,
)
from vllm.tokenizers import get_tokenizer
from vllm.utils.serial_utils import EMBED_DTYPES, ENDIANNESS, binary2tensor
```
**EN:** Imports standard-library modules such as `json`, third-party packages like `numpy`, `pybase64`, `pytest`, project helpers such as `tests.models.utils.check_embeddings_close`, `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.pooling.pooling.protocol.PoolingResponse`.
**CN:** 导入标准库模块（如 `json`）、第三方包（如 `numpy`、`pybase64`、`pytest`）、项目内辅助模块（如 `tests.models.utils.check_embeddings_close`、`tests.utils.RemoteOpenAIServer`、`vllm.entrypoints.pooling.pooling.protocol.PoolingResponse`）。

### Module setup / 模块级配置: MODEL_NAME, DUMMY_CHAT_TEMPLATE, input_text (L23-L26)
```python
MODEL_NAME = "internlm/internlm2-1_8b-reward"
DUMMY_CHAT_TEMPLATE = """{% for message in messages %}{{message['role'] + ': ' + message['content'] + '\\n'}}{% endfor %}"""  # noqa: E501
input_text = "The chef prepared a delicious meal."
input_tokens = [1, 918, 29981, 10166, 395, 18067, 15265, 281]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `DUMMY_CHAT_TEMPLATE`, `input_text`, `input_tokens`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`DUMMY_CHAT_TEMPLATE`、`input_text`、`input_tokens`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L29-L46)
```python
@pytest.fixture(scope="module")
def server():
    args = [
        "--runner",
        "pooling",
        # use half precision for speed and memory savings in CI environment
        "--dtype",
        "bfloat16",
        "--enforce-eager",
        "--max-model-len",
        "512",
        "--chat-template",
        DUMMY_CHAT_TEMPLATE,
        "--trust-remote-code",
    ]

    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_basic (L49-L61)
```python
@pytest.mark.parametrize("model_name", [MODEL_NAME])
def test_basic(server: RemoteOpenAIServer, model_name: str):
    # test /v1/models
    response = requests.get(server.url_for("/v1/models"))
    served_model = response.json()["data"][0]["id"]
    assert served_model == MODEL_NAME

    # test /tokenize
    response = requests.post(
        server.url_for("/tokenize"),
        json={"model": model_name, "prompt": input_text},
    )
    assert response.json()["tokens"] == input_tokens
```
**EN:** This test validates `test_basic`. It uses parameterization over `model_name`. Key inputs are `server`, `model_name`. The main assertion is `served_model == MODEL_NAME` and `response.json()['tokens'] == input_tokens`.
**CN:** 这个测试验证 `test_basic`。 它通过参数化组合 `model_name`。 关键输入包括 `server`、`model_name`。 核心断言是 `served_model == MODEL_NAME` and `response.json()['tokens'] == input_tokens`。

### Test / 测试: test_completion_request (L64-L95)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
def test_completion_request(server: RemoteOpenAIServer, model_name: str):
    # test input: str
    response = requests.post(
        server.url_for("pooling"),
        json={"model": model_name, "input": input_text, "encoding_format": "float"},
    )
    response.raise_for_status()
    poolings = PoolingResponse.model_validate(response.json())

    assert poolings.id is not None
    assert len(poolings.data) == 1
    assert len(poolings.data[0].data) == len(input_tokens)
    assert poolings.usage.completion_tokens == 0
    assert poolings.usage.prompt_tokens == len(input_tokens)
    assert poolings.usage.total_tokens == len(input_tokens)

# ... 6 lines omitted for brevity ...
    poolings = PoolingResponse.model_validate(response.json())

    assert poolings.id is not None
    assert len(poolings.data) == 1
    assert len(poolings.data[0].data) == len(input_tokens)
    assert poolings.usage.completion_tokens == 0
    assert poolings.usage.prompt_tokens == len(input_tokens)
    assert poolings.usage.total_tokens == len(input_tokens)
```
**EN:** This test validates `test_completion_request`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `model_name`. The main assertion is `poolings.id is not None` and `len(poolings.data) == 1`.
**CN:** 这个测试验证 `test_completion_request`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`model_name`。 核心断言是 `poolings.id is not None` and `len(poolings.data) == 1`。

### Test / 测试: test_completion_request_batched (L98-L134)
```python
@pytest.mark.parametrize("model_name", [MODEL_NAME])
def test_completion_request_batched(server: RemoteOpenAIServer, model_name: str):
    N = 10
    input_texts = [input_text] * N

    response = requests.post(
        server.url_for("pooling"),
        json={"model": model_name, "input": input_texts, "encoding_format": "float"},
    )
    response.raise_for_status()
    poolings = PoolingResponse.model_validate(response.json())

    assert poolings.id is not None
    assert len(poolings.data) == N
    assert len(poolings.data[0].data) == len(input_tokens)
    assert poolings.usage.completion_tokens == 0
    assert poolings.usage.prompt_tokens == len(input_tokens) * N
    assert poolings.usage.total_tokens == len(input_tokens) * N
# ... 11 lines omitted for brevity ...
    poolings = PoolingResponse.model_validate(response.json())

    assert poolings.id is not None
    assert len(poolings.data) == N
    assert len(poolings.data[0].data) == len(input_tokens)
    assert poolings.usage.completion_tokens == 0
    assert poolings.usage.prompt_tokens == len(input_tokens) * N
    assert poolings.usage.total_tokens == len(input_tokens) * N
```
**EN:** This test validates `test_completion_request_batched`. It uses parameterization over `model_name`. Key inputs are `server`, `model_name`. The main assertion is `poolings.id is not None` and `len(poolings.data) == N`.
**CN:** 这个测试验证 `test_completion_request_batched`。 它通过参数化组合 `model_name`。 关键输入包括 `server`、`model_name`。 核心断言是 `poolings.id is not None` and `len(poolings.data) == N`。

### Test / 测试: test_chat_request (L137-L255)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_chat_request(server: RemoteOpenAIServer, model_name: str):
    messages = [
        {
            "role": "user",
            "content": "The cat sat on the mat.",
        },
        {
            "role": "assistant",
            "content": "A feline was resting on a rug.",
        },
        {
            "role": "user",
            "content": "Stars twinkle brightly in the night sky.",
        },
    ]

# ... 93 lines omitted for brevity ...
            "continue_final_message": True,
            "add_generation_prompt": True,
        },
    )
    assert (
        "Cannot set both `continue_final_message` and `add_generation_prompt` to True."
        in response.json()["error"]["message"]
    )
```
**EN:** This async test validates `test_chat_request`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `model_name`. The main assertion is `chat_poolings.id is not None` and `completion_poolings.id is not None`.
**CN:** 这个异步测试验证 `test_chat_request`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`model_name`。 核心断言是 `chat_poolings.id is not None` and `completion_poolings.id is not None`。

### Test / 测试: test_batch_base64_pooling (L258-L321)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_batch_base64_pooling(server: RemoteOpenAIServer, model_name: str):
    input_texts = [
        "Hello my name is",
        "The best thing about vLLM is that it supports many different models",
    ]

    float_response = requests.post(
        server.url_for("pooling"),
        json={
            "input": input_texts,
            "model": model_name,
            "encoding_format": "float",
        },
    )
    float_response.raise_for_status()
    responses_float = PoolingResponse.model_validate(float_response.json())
# ... 38 lines omitted for brevity ...
    ]

    check_embeddings_close(
        embeddings_0_lst=float_data,
        embeddings_1_lst=default_data,
        name_0="float32",
        name_1="default",
    )
```
**EN:** This async test validates `test_batch_base64_pooling`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `model_name`.
**CN:** 这个异步测试验证 `test_batch_base64_pooling`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`model_name`。

### Test / 测试: test_base64_embed_dtype_and_endianness (L324-L368)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_base64_embed_dtype_and_endianness(
    server: RemoteOpenAIServer, model_name: str
):
    input_texts = [input_text] * 3

    url = server.url_for("pooling")
    float_response = requests.post(
        url,
        json={
            "model": model_name,
            "input": input_texts,
            "encoding_format": "float",
        },
    )
    responses_float = PoolingResponse.model_validate(float_response.json())
    float_data = [np.array(d.data).squeeze(-1).tolist() for d in responses_float.data]
# ... 19 lines omitted for brevity ...

            check_embeddings_close(
                embeddings_0_lst=float_data,
                embeddings_1_lst=base64_data,
                name_0="float_data",
                name_1="base64_data",
                tol=1e-2,
            )
```
**EN:** This async test validates `test_base64_embed_dtype_and_endianness`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `model_name`.
**CN:** 这个异步测试验证 `test_base64_embed_dtype_and_endianness`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`model_name`。

### Test / 测试: test_bytes_embed_dtype_and_endianness (L371-L416)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_bytes_embed_dtype_and_endianness(
    server: RemoteOpenAIServer, model_name: str
):
    input_texts = [input_text] * 3

    url = server.url_for("pooling")
    float_response = requests.post(
        url,
        json={
            "model": model_name,
            "input": input_texts,
            "encoding_format": "float",
        },
    )
    responses_float = PoolingResponse.model_validate(float_response.json())
    float_data = [np.array(d.data).squeeze(-1).tolist() for d in responses_float.data]
# ... 20 lines omitted for brevity ...

            check_embeddings_close(
                embeddings_0_lst=float_data,
                embeddings_1_lst=bytes_data,
                name_0="float_data",
                name_1="bytes_data",
                tol=1e-2,
            )
```
**EN:** This async test validates `test_bytes_embed_dtype_and_endianness`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `model_name`.
**CN:** 这个异步测试验证 `test_bytes_embed_dtype_and_endianness`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`model_name`。

### Test / 测试: test_bytes_only_embed_dtype_and_endianness (L419-L469)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_bytes_only_embed_dtype_and_endianness(
    server: RemoteOpenAIServer, model_name: str
):
    input_texts = [input_text] * 3

    url = server.url_for("pooling")
    float_response = requests.post(
        url,
        json={
            "model": model_name,
            "input": input_texts,
            "encoding_format": "float",
        },
    )
    responses_float = PoolingResponse.model_validate(float_response.json())
    float_data = [np.array(d.data).squeeze(-1).tolist() for d in responses_float.data]
# ... 25 lines omitted for brevity ...

            check_embeddings_close(
                embeddings_0_lst=float_data,
                embeddings_1_lst=bytes_data,
                name_0="float_data",
                name_1="bytes_data",
                tol=1e-2,
            )
```
**EN:** This async test validates `test_bytes_only_embed_dtype_and_endianness`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `model_name`. The main assertion is `'metadata' not in responses_bytes.headers`.
**CN:** 这个异步测试验证 `test_bytes_only_embed_dtype_and_endianness`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`model_name`。 核心断言是 `'metadata' not in responses_bytes.headers`。

### Test / 测试: test_params_not_supported (L472-L490)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("param_name", ["encoding_format", "embed_dtype", "endianness"])
async def test_params_not_supported(
    server: RemoteOpenAIServer, model_name: str, param_name: str
):
    responses_base64 = requests.post(
        server.url_for("pooling"),
        json={
            "model": model_name,
            "input": input_text,
            "encoding_format": "base64",
            param_name: f"bad_{param_name}",
        },
    )

    assert responses_base64.status_code == 400
    assert "literal_error" in responses_base64.json()["error"]["message"]
    assert f"bad_{param_name}" in responses_base64.json()["error"]["message"]
```
**EN:** This async test validates `test_params_not_supported`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `model_name`, `param_name`. The main assertion is `responses_base64.status_code == 400` and `'literal_error' in responses_base64.json()['error']['message']`.
**CN:** 这个异步测试验证 `test_params_not_supported`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`model_name`、`param_name`。 核心断言是 `responses_base64.status_code == 400` and `'literal_error' in responses_base64.json()['error']['message']`。

### Test / 测试: test_invocations_chat_request (L493-L522)
```python
@pytest.mark.asyncio
async def test_invocations_chat_request(server: RemoteOpenAIServer):
    request_args = {
        "model": MODEL_NAME,
        "input": input_text,
        "encoding_format": "float",
    }

    completion_response = requests.post(server.url_for("pooling"), json=request_args)
    completion_response.raise_for_status()

    invocation_response = requests.post(
        server.url_for("invocations"), json=request_args
    )
    invocation_response.raise_for_status()

    completion_output = completion_response.json()
    invocation_output = invocation_response.json()

    assert completion_output.keys() == invocation_output.keys()
    for completion_data, invocation_data in zip(
        completion_output["data"], invocation_output["data"]
    ):
        assert completion_data.keys() == invocation_data.keys()
        check_embeddings_close(
            embeddings_0_lst=completion_data["data"],
            embeddings_1_lst=invocation_data["data"],
            name_0="completion",
            name_1="invocation",
        )
```
**EN:** This async test validates `test_invocations_chat_request`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `completion_output.keys() == invocation_output.keys()` and `completion_data.keys() == invocation_data.keys()`.
**CN:** 这个异步测试验证 `test_invocations_chat_request`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `completion_output.keys() == invocation_output.keys()` and `completion_data.keys() == invocation_data.keys()`。

### Test / 测试: test_invocations_conversation_chat_request (L525-L569)
```python
@pytest.mark.asyncio
async def test_invocations_conversation_chat_request(server: RemoteOpenAIServer):
    messages = [
        {
            "role": "user",
            "content": "The cat sat on the mat.",
        },
        {
            "role": "assistant",
            "content": "A feline was resting on a rug.",
        },
        {
            "role": "user",
            "content": "Stars twinkle brightly in the night sky.",
        },
    ]

    request_args = {
# ... 19 lines omitted for brevity ...
    ):
        assert chat_data.keys() == invocation_data.keys()
        check_embeddings_close(
            embeddings_0_lst=chat_data["data"],
            embeddings_1_lst=invocation_data["data"],
            name_0="chat",
            name_1="invocation",
        )
```
**EN:** This async test validates `test_invocations_conversation_chat_request`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `chat_output.keys() == invocation_output.keys()` and `chat_data.keys() == invocation_data.keys()`.
**CN:** 这个异步测试验证 `test_invocations_conversation_chat_request`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `chat_output.keys() == invocation_output.keys()` and `chat_data.keys() == invocation_data.keys()`。

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
- **Stdlib / 标准库**: `json`
- **Third-party / 第三方**: `numpy`, `pybase64`, `pytest`, `requests`, `torch`
- **Project / 项目内**: `tests.models.utils.check_embeddings_close`, `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.pooling.pooling.protocol.PoolingResponse`, `vllm.entrypoints.pooling.utils.MetadataItem`, `vllm.entrypoints.pooling.utils.build_metadata_items`, `vllm.entrypoints.pooling.utils.decode_pooling_output`, `vllm.tokenizers.get_tokenizer`, `vllm.utils.serial_utils.EMBED_DTYPES`, `vllm.utils.serial_utils.ENDIANNESS`, `vllm.utils.serial_utils.binary2tensor`
