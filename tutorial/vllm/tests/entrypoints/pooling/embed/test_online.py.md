# test_online.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/embed/test_online.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior. The file defines 15 test(s), 3 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为。它定义了 15 个测试、3 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L27)
```python
import json

import numpy as np
import openai
import pybase64 as base64
import pytest
import pytest_asyncio
import requests
import torch
import torch.nn.functional as F

from tests.models.language.pooling.embed_utils import run_embedding_correctness_test
from tests.models.utils import check_embeddings_close
from tests.utils import RemoteOpenAIServer
from vllm.entrypoints.pooling.embed.protocol import EmbeddingResponse
from vllm.entrypoints.pooling.pooling.protocol import PoolingResponse
from vllm.entrypoints.pooling.utils import (
    MetadataItem,
    build_metadata_items,
    decode_pooling_output,
)
from vllm.platforms import current_platform
from vllm.tokenizers import get_tokenizer
from vllm.utils.serial_utils import EMBED_DTYPES, ENDIANNESS, binary2tensor
```
**EN:** Imports standard-library modules such as `json`, third-party packages like `numpy`, `openai`, `pybase64`, project helpers such as `tests.models.language.pooling.embed_utils.run_embedding_correctness_test`, `tests.models.utils.check_embeddings_close`, `tests.utils.RemoteOpenAIServer`.
**CN:** 导入标准库模块（如 `json`）、第三方包（如 `numpy`、`openai`、`pybase64`）、项目内辅助模块（如 `tests.models.language.pooling.embed_utils.run_embedding_correctness_test`、`tests.models.utils.check_embeddings_close`、`tests.utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: MODEL_NAME, DUMMY_CHAT_TEMPLATE, DTYPE (L29-L66)
```python
MODEL_NAME = "intfloat/multilingual-e5-small"
DUMMY_CHAT_TEMPLATE = """{% for message in messages %}{{message['role'] + ': ' + message['content'] + '\\n'}}{% endfor %}"""  # noqa: E501
DTYPE = "bfloat16"
input_text = "The best thing about vLLM is that it supports many different models"
input_tokens = [
    0,
    581,
    2965,
    13580,
    1672,
    81,
    23708,
    594,
    83,
    450,
    442,
# ... 16 lines omitted for brevity ...
# On ROCm, floating-point reductions in attention and GEMM kernels are
# non-associative and sensitive to batch geometry. Force LLM instances
# into an identical, deterministic execution mode:
ROCM_DETERMINISM_ARGS: list[str] = (
    ["--max-num-seqs", "1"] if current_platform.is_rocm() else []
)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `DUMMY_CHAT_TEMPLATE`, `DTYPE`, `input_text`, `input_tokens`, `ROCM_DETERMINISM_ARGS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`DUMMY_CHAT_TEMPLATE`、`DTYPE`、`input_text`、`input_tokens`、`ROCM_DETERMINISM_ARGS`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L69-L85)
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
        "--chat-template",
        DUMMY_CHAT_TEMPLATE,
        *ROCM_DETERMINISM_ARGS,
    ]

    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L88-L91)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: hf_model (L94-L97)
```python
@pytest.fixture(scope="module")
def hf_model(hf_runner):
    with hf_runner(MODEL_NAME, dtype=DTYPE, is_sentence_transformer=True) as hf_model:
        yield hf_model
```
**EN:** This fixture prepares `hf_model` for dependent tests. Key inputs are `hf_runner`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `hf_model`。 关键输入包括 `hf_runner`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_basic (L100-L120)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_basic(
    server: RemoteOpenAIServer, client: openai.AsyncOpenAI, model_name: str
):
    # test /v1/models
    response = requests.get(server.url_for("/v1/models"))
    model = response.json()["data"][0]["id"]
    assert model == MODEL_NAME

    models = await client.models.list()
    models = models.data
    served_model = models[0]
    assert served_model.id == MODEL_NAME

    # test /tokenize
    response = requests.post(
        server.url_for("/tokenize"),
        json={"model": model_name, "prompt": input_text},
    )
    assert response.json()["tokens"] == input_tokens
```
**EN:** This async test validates `test_basic`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `client`, `model_name`. The main assertion is `model == MODEL_NAME` and `served_model.id == MODEL_NAME`.
**CN:** 这个异步测试验证 `test_basic`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`client`、`model_name`。 核心断言是 `model == MODEL_NAME` and `served_model.id == MODEL_NAME`。

### Test / 测试: test_completion_request (L123-L166)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_completion_request(
    client: openai.AsyncOpenAI, model_name: str, hf_model
):
    # test input: str
    embedding_response = await client.embeddings.create(
        model=model_name,
        input=input_text,
        encoding_format="float",
    )
    embeddings = EmbeddingResponse.model_validate(
        embedding_response.model_dump(mode="json")
    )

    assert embeddings.id is not None
    assert len(embeddings.data) == 1
    assert len(embeddings.data[0].embedding) == 384
# ... 18 lines omitted for brevity ...
    assert len(embeddings.data) == 1
    assert len(embeddings.data[0].embedding) == 384
    assert embeddings.usage.completion_tokens == 0
    assert embeddings.usage.prompt_tokens == len(input_tokens)
    assert embeddings.usage.total_tokens == len(input_tokens)

    vllm_outputs = [d.embedding for d in embeddings.data]
    run_embedding_correctness_test(hf_model, [input_text], vllm_outputs)
```
**EN:** This async test validates `test_completion_request`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `hf_model`. It drives client-facing request creation through the API surface under test. The main assertion is `embeddings.id is not None` and `len(embeddings.data) == 1`.
**CN:** 这个异步测试验证 `test_completion_request`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`hf_model`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `embeddings.id is not None` and `len(embeddings.data) == 1`。

### Test / 测试: test_completion_request_batched (L169-L215)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_completion_request_batched(
    client: openai.AsyncOpenAI, model_name: str, hf_model
):
    N = 10
    input_texts = [input_text] * N

    # test input: list[str]
    embedding_response = await client.embeddings.create(
        model=model_name,
        input=input_texts,
        encoding_format="float",
    )
    embeddings = EmbeddingResponse.model_validate(
        embedding_response.model_dump(mode="json")
    )

# ... 21 lines omitted for brevity ...
    assert len(embeddings.data) == N
    assert len(embeddings.data[0].embedding) == 384
    assert embeddings.usage.completion_tokens == 0
    assert embeddings.usage.prompt_tokens == len(input_tokens) * N
    assert embeddings.usage.total_tokens == len(input_tokens) * N

    vllm_outputs = [d.embedding for d in embeddings.data]
    run_embedding_correctness_test(hf_model, input_texts, vllm_outputs)
```
**EN:** This async test validates `test_completion_request_batched`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `hf_model`. It drives client-facing request creation through the API surface under test. The main assertion is `embeddings.id is not None` and `len(embeddings.data) == N`.
**CN:** 这个异步测试验证 `test_completion_request_batched`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`hf_model`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `embeddings.id is not None` and `len(embeddings.data) == N`。

### Test / 测试: test_truncate_prompt_tokens (L218-L292)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_truncate_prompt_tokens(client: openai.AsyncOpenAI, model_name: str):
    input_texts = [
        "Como o Brasil pode fomentar o desenvolvimento de modelos de IA?",
    ]

    # test single embedding
    embedding_response = await client.embeddings.create(
        model=model_name, input=input_texts, extra_body={"truncate_prompt_tokens": 10}
    )
    embeddings = EmbeddingResponse.model_validate(
        embedding_response.model_dump(mode="json")
    )

    assert embeddings.id is not None
    assert len(embeddings.data) == 1
    assert len(embeddings.data[0].embedding) == 384
# ... 49 lines omitted for brevity ...
            input=input_texts,
            extra_body={"truncate_prompt_tokens": 8193},
        )
        assert "error" in response.object
        assert (
            "truncate_prompt_tokens value is greater than max_model_len. "
            "Please request a smaller truncation size." in response.message
        )
```
**EN:** This async test validates `test_truncate_prompt_tokens`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `embeddings.id is not None` and `len(embeddings.data) == 1`.
**CN:** 这个异步测试验证 `test_truncate_prompt_tokens`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `embeddings.id is not None` and `len(embeddings.data) == 1`。

### Test / 测试: test_chat_request (L295-L419)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_chat_request(
    server: RemoteOpenAIServer, client: openai.AsyncOpenAI, model_name: str
):
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
# ... 99 lines omitted for brevity ...
            "continue_final_message": True,
            "add_generation_prompt": True,
        },
    )
    assert (
        "Cannot set both `continue_final_message` and `add_generation_prompt` to True."
        in response.json()["error"]["message"]
    )
```
**EN:** This async test validates `test_chat_request`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `chat_embeddings.id is not None` and `completion_embeddings.id is not None`.
**CN:** 这个异步测试验证 `test_chat_request`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `chat_embeddings.id is not None` and `completion_embeddings.id is not None`。

### Test / 测试: test_invocations_completion_request (L422-L452)
```python
@pytest.mark.asyncio
async def test_invocations_completion_request(
    server: RemoteOpenAIServer, client: openai.AsyncOpenAI
):
    request_args = {
        "model": MODEL_NAME,
        "input": input_text,
        "encoding_format": "float",
    }

    completion_response = await client.embeddings.create(**request_args)

    invocation_response = requests.post(
        server.url_for("invocations"), json=request_args
    )
    invocation_response.raise_for_status()

    completion_output = completion_response.model_dump()
# ... 5 lines omitted for brevity ...
    ):
        assert completion_data.keys() == invocation_data.keys()
        check_embeddings_close(
            embeddings_0_lst=[completion_data["embedding"]],
            embeddings_1_lst=[invocation_data["embedding"]],
            name_0="completion",
            name_1="invocation",
        )
```
**EN:** This async test validates `test_invocations_completion_request`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `client`. It drives client-facing request creation through the API surface under test. The main assertion is `completion_output.keys() == invocation_output.keys()` and `completion_data.keys() == invocation_data.keys()`.
**CN:** 这个异步测试验证 `test_invocations_completion_request`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `completion_output.keys() == invocation_output.keys()` and `completion_data.keys() == invocation_data.keys()`。

### Test / 测试: test_invocations_chat_request (L455-L499)
```python
@pytest.mark.asyncio
async def test_invocations_chat_request(server: RemoteOpenAIServer):
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
            embeddings_0_lst=[chat_data["embedding"]],
            embeddings_1_lst=[invocation_data["embedding"]],
            name_0="chat",
            name_1="invocation",
        )
```
**EN:** This async test validates `test_invocations_chat_request`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `chat_output.keys() == invocation_output.keys()` and `chat_data.keys() == invocation_data.keys()`.
**CN:** 这个异步测试验证 `test_invocations_chat_request`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `chat_output.keys() == invocation_output.keys()` and `chat_data.keys() == invocation_data.keys()`。

### Test / 测试: test_base64_embedding (L502-L532)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_base64_embedding(hf_model, client: openai.AsyncOpenAI, model_name: str):
    input_texts = [
        "Hello my name is",
        "The best thing about vLLM is that it supports many different models",
    ]

    responses_float = await client.embeddings.create(
        input=input_texts, model=model_name, encoding_format="float"
    )
    float_data = [d.embedding for d in responses_float.data]
    run_embedding_correctness_test(hf_model, input_texts, float_data)

    responses_base64 = await client.embeddings.create(
        input=input_texts, model=model_name, encoding_format="base64"
    )
    base64_data = []
# ... 5 lines omitted for brevity ...
    run_embedding_correctness_test(hf_model, input_texts, base64_data)

    # Default response is float32 decoded from base64 by OpenAI Client
    responses_default = await client.embeddings.create(
        input=input_texts, model=model_name
    )
    default_data = [d.embedding for d in responses_default.data]
    run_embedding_correctness_test(hf_model, input_texts, default_data)
```
**EN:** This async test validates `test_base64_embedding`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `hf_model`, `client`, `model_name`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_base64_embedding`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `hf_model`、`client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_base64_embed_dtype_and_endianness (L535-L571)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_base64_embed_dtype_and_endianness(
    server: RemoteOpenAIServer, client: openai.AsyncOpenAI, model_name: str
):
    input_texts = [input_text] * 3
    responses_float = await client.embeddings.create(
        input=input_texts, model=model_name, encoding_format="float"
    )
    float_data = [d.embedding for d in responses_float.data]

    for embed_dtype in EMBED_DTYPES:
        for endianness in ENDIANNESS:
            responses_base64 = requests.post(
                server.url_for("/v1/embeddings"),
                json={
                    "model": model_name,
                    "input": input_texts,
# ... 11 lines omitted for brevity ...

            check_embeddings_close(
                embeddings_0_lst=float_data,
                embeddings_1_lst=base64_data,
                name_0="float_data",
                name_1="base64_data",
                tol=1e-2,
            )
```
**EN:** This async test validates `test_base64_embed_dtype_and_endianness`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `client`, `model_name`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_base64_embed_dtype_and_endianness`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_bytes_embed_dtype_and_endianness (L574-L611)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_bytes_embed_dtype_and_endianness(
    server: RemoteOpenAIServer, client: openai.AsyncOpenAI, model_name: str
):
    input_texts = [input_text] * 3
    responses_float = await client.embeddings.create(
        input=input_texts, model=model_name, encoding_format="float"
    )
    float_data = [d.embedding for d in responses_float.data]

    for embed_dtype in EMBED_DTYPES:
        for endianness in ENDIANNESS:
            responses_bytes = requests.post(
                server.url_for("/v1/embeddings"),
                json={
                    "model": model_name,
                    "input": input_texts,
# ... 12 lines omitted for brevity ...

            check_embeddings_close(
                embeddings_0_lst=float_data,
                embeddings_1_lst=bytes_data,
                name_0="float_data",
                name_1="bytes_data",
                tol=1e-2,
            )
```
**EN:** This async test validates `test_bytes_embed_dtype_and_endianness`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `client`, `model_name`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_bytes_embed_dtype_and_endianness`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_bytes_only_embed_dtype_and_endianness (L614-L660)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_bytes_only_embed_dtype_and_endianness(
    server: RemoteOpenAIServer, client: openai.AsyncOpenAI, model_name: str
):
    input_texts = [
        "The best thing about vLLM is that it supports many different models",
    ] * 2

    responses_float = await client.embeddings.create(
        input=input_texts, model=model_name, encoding_format="float"
    )
    float_data = [d.embedding for d in responses_float.data]
    embedding_size = len(float_data[0])

    for embed_dtype in EMBED_DTYPES:
        for endianness in ENDIANNESS:
            responses_bytes = requests.post(
# ... 21 lines omitted for brevity ...

            check_embeddings_close(
                embeddings_0_lst=float_data,
                embeddings_1_lst=bytes_data,
                name_0="float_data",
                name_1="bytes_data",
                tol=1e-2,
            )
```
**EN:** This async test validates `test_bytes_only_embed_dtype_and_endianness`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `'metadata' not in responses_bytes.headers`.
**CN:** 这个异步测试验证 `test_bytes_only_embed_dtype_and_endianness`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `'metadata' not in responses_bytes.headers`。

### Test / 测试: test_params_not_supported (L663-L681)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("param_name", ["encoding_format", "embed_dtype", "endianness"])
async def test_params_not_supported(
    server: RemoteOpenAIServer, model_name: str, param_name: str
):
    responses_base64 = requests.post(
        server.url_for("/v1/embeddings"),
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

### Test / 测试: test_use_activation (L684-L710)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_use_activation(server: RemoteOpenAIServer, model_name: str):
    async def get_outputs(use_activation):
        request_args = {
            "model": MODEL_NAME,
            "input": input_text,
            "encoding_format": "float",
            "use_activation": use_activation,
        }

        response = requests.post(server.url_for("v1/embeddings"), json=request_args)
        outputs = response.json()

        return torch.tensor([x["embedding"] for x in outputs["data"]])

    default = await get_outputs(use_activation=None)
    w_normal = await get_outputs(use_activation=True)
    wo_normal = await get_outputs(use_activation=False)

    assert torch.allclose(default, w_normal, atol=1e-2), "Default should use normal."
    assert not torch.allclose(w_normal, wo_normal, atol=1e-2), (
        "wo_normal should not use normal."
    )
    assert torch.allclose(w_normal, F.normalize(wo_normal, p=2, dim=-1), atol=1e-2), (
        "w_normal should be close to normal(wo_normal)."
    )
```
**EN:** This async test validates `test_use_activation`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `model_name`. The main assertion is `torch.allclose(default, w_normal, atol=0.01)` and `not torch.allclose(w_normal, wo_normal, atol=0.01)`.
**CN:** 这个异步测试验证 `test_use_activation`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`model_name`。 核心断言是 `torch.allclose(default, w_normal, atol=0.01)` and `not torch.allclose(w_normal, wo_normal, atol=0.01)`。

### Test / 测试: test_pooling_embed (L713-L730)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_pooling_embed(server: RemoteOpenAIServer, model_name: str):
    task = "embed"
    response = requests.post(
        server.url_for("pooling"),
        json={
            "model": model_name,
            "input": input_text,
            "encoding_format": "float",
            "task": task,
        },
    )

    poolings = PoolingResponse.model_validate(response.json())

    assert len(poolings.data) == 1
    assert len(poolings.data[0].data) == 384
```
**EN:** This async test validates `test_pooling_embed`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `model_name`. The main assertion is `len(poolings.data) == 1` and `len(poolings.data[0].data) == 384`.
**CN:** 这个异步测试验证 `test_pooling_embed`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`model_name`。 核心断言是 `len(poolings.data) == 1` and `len(poolings.data[0].data) == 384`。

### Test / 测试: test_pooling_not_supported (L733-L757)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize(
    "task", ["classify", "token_classify", "token_embed", "plugin"]
)
async def test_pooling_not_supported(
    server: RemoteOpenAIServer, model_name: str, task: str
):
    response = requests.post(
        server.url_for("pooling"),
        json={
            "model": model_name,
            "input": "test",
            "encoding_format": "float",
            "task": task,
        },
    )
    assert response.json()["error"]["type"] == "BadRequestError"
    if task == "plugin":
        err_msg = "No IOProcessor plugin installed."
    elif task == "token_embed":
        err_msg = "Try switching the model's pooling_task via"
    else:
        err_msg = f"Unsupported task: {task!r}"
    assert response.json()["error"]["message"].startswith(err_msg)
```
**EN:** This async test validates `test_pooling_not_supported`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `model_name`, `task`. The main assertion is `response.json()['error']['type'] == 'BadRequestError'` and `response.json()['error']['message'].startswith(err_msg)`.
**CN:** 这个异步测试验证 `test_pooling_not_supported`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`model_name`、`task`。 核心断言是 `response.json()['error']['type'] == 'BadRequestError'` and `response.json()['error']['message'].startswith(err_msg)`。

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
- **Third-party / 第三方**: `numpy`, `openai`, `pybase64`, `pytest`, `pytest_asyncio`, `requests`, `torch`, `torch.nn.functional`
- **Project / 项目内**: `tests.models.language.pooling.embed_utils.run_embedding_correctness_test`, `tests.models.utils.check_embeddings_close`, `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.pooling.embed.protocol.EmbeddingResponse`, `vllm.entrypoints.pooling.pooling.protocol.PoolingResponse`, `vllm.entrypoints.pooling.utils.MetadataItem`, `vllm.entrypoints.pooling.utils.build_metadata_items`, `vllm.entrypoints.pooling.utils.decode_pooling_output`, `vllm.platforms.current_platform`, `vllm.tokenizers.get_tokenizer`, `vllm.utils.serial_utils.EMBED_DTYPES`, `vllm.utils.serial_utils.ENDIANNESS`, `vllm.utils.serial_utils.binary2tensor`
