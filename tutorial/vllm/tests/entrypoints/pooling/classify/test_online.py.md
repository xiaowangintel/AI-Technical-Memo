# test_online.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/classify/test_online.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior. The file defines 14 test(s), 1 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为。它定义了 14 个测试、1 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L11)
```python
import pytest
import requests
import torch
import torch.nn.functional as F

from tests.utils import RemoteOpenAIServer
from vllm.entrypoints.pooling.classify.protocol import ClassificationResponse
from vllm.entrypoints.pooling.pooling.protocol import PoolingResponse
```
**EN:** Imports third-party packages like `pytest`, `requests`, `torch`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.pooling.classify.protocol.ClassificationResponse`, `vllm.entrypoints.pooling.pooling.protocol.PoolingResponse`.
**CN:** 导入第三方包（如 `pytest`、`requests`、`torch`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.entrypoints.pooling.classify.protocol.ClassificationResponse`、`vllm.entrypoints.pooling.pooling.protocol.PoolingResponse`）。

### Module setup / 模块级配置: MODEL_NAME, DTYPE, input_text (L13-L16)
```python
MODEL_NAME = "jason9693/Qwen2.5-1.5B-apeach"
DTYPE = "float32"  # Use float32 to avoid NaN issue
input_text = "This product was excellent and exceeded my expectations"
input_tokens = [1986, 1985, 572, 9073, 323, 33808, 847, 16665]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `DTYPE`, `input_text`, `input_tokens`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`DTYPE`、`input_text`、`input_tokens`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L19-L30)
```python
@pytest.fixture(scope="module")
def server():
    args = [
        "--enforce-eager",
        "--max-model-len",
        "512",
        "--dtype",
        DTYPE,
    ]

    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_basic (L33-L45)
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

### Test / 测试: test_completion_request (L48-L78)
```python
@pytest.mark.parametrize("model_name", [MODEL_NAME])
def test_completion_request(server: RemoteOpenAIServer, model_name: str):
    # test input: str
    classification_response = requests.post(
        server.url_for("classify"),
        json={"model": model_name, "input": input_text},
    )

    classification_response.raise_for_status()
    output = ClassificationResponse.model_validate(classification_response.json())

    assert output.object == "list"
    assert output.model == MODEL_NAME
    assert len(output.data) == 1
    assert hasattr(output.data[0], "label")
    assert hasattr(output.data[0], "probs")

    # test input: list[int]
# ... 5 lines omitted for brevity ...
    classification_response.raise_for_status()
    output = ClassificationResponse.model_validate(classification_response.json())

    assert output.object == "list"
    assert output.model == MODEL_NAME
    assert len(output.data) == 1
    assert hasattr(output.data[0], "label")
    assert hasattr(output.data[0], "probs")
```
**EN:** This test validates `test_completion_request`. It uses parameterization over `model_name`. Key inputs are `server`, `model_name`. The main assertion is `output.object == 'list'` and `output.model == MODEL_NAME`.
**CN:** 这个测试验证 `test_completion_request`。 它通过参数化组合 `model_name`。 关键输入包括 `server`、`model_name`。 核心断言是 `output.object == 'list'` and `output.model == MODEL_NAME`。

### Test / 测试: test_completion_request_batched (L81-L113)
```python
@pytest.mark.parametrize("model_name", [MODEL_NAME])
def test_completion_request_batched(server: RemoteOpenAIServer, model_name: str):
    N = 10

    # test input: list[str]
    classification_response = requests.post(
        server.url_for("classify"),
        json={"model": model_name, "input": [input_text] * N},
    )
    output = ClassificationResponse.model_validate(classification_response.json())

    assert len(output.data) == N
    for i, item in enumerate(output.data):
        assert item.index == i
        assert hasattr(item, "label")
        assert hasattr(item, "probs")
        assert len(item.probs) == item.num_classes
        assert item.label in ["Default", "Spoiled"]
# ... 7 lines omitted for brevity ...

    assert len(output.data) == N
    for i, item in enumerate(output.data):
        assert item.index == i
        assert hasattr(item, "label")
        assert hasattr(item, "probs")
        assert len(item.probs) == item.num_classes
        assert item.label in ["Default", "Spoiled"]
```
**EN:** This test validates `test_completion_request_batched`. It uses parameterization over `model_name`. Key inputs are `server`, `model_name`. The main assertion is `len(output.data) == N` and `len(output.data) == N`.
**CN:** 这个测试验证 `test_completion_request_batched`。 它通过参数化组合 `model_name`。 关键输入包括 `server`、`model_name`。 核心断言是 `len(output.data) == N` and `len(output.data) == N`。

### Test / 测试: test_empty_input_error (L116-L134)
```python
@pytest.mark.parametrize("model_name", [MODEL_NAME])
def test_empty_input_error(server: RemoteOpenAIServer, model_name: str):
    classification_response = requests.post(
        server.url_for("classify"),
        json={"model": model_name, "input": ""},
    )

    error = classification_response.json()
    assert classification_response.status_code == 400
    assert "error" in error

    classification_response = requests.post(
        server.url_for("classify"),
        json={"model": model_name, "input": []},
    )

    error = classification_response.json()
    assert classification_response.status_code == 400
    assert "error" in error
```
**EN:** This test validates `test_empty_input_error`. It uses parameterization over `model_name`. Key inputs are `server`, `model_name`. The main assertion is `classification_response.status_code == 400` and `'error' in error`.
**CN:** 这个测试验证 `test_empty_input_error`。 它通过参数化组合 `model_name`。 关键输入包括 `server`、`model_name`。 核心断言是 `classification_response.status_code == 400` and `'error' in error`。

### Test / 测试: test_truncate_prompt_tokens (L137-L163)
```python
@pytest.mark.parametrize("model_name", [MODEL_NAME])
def test_truncate_prompt_tokens(server: RemoteOpenAIServer, model_name: str):
    long_text = "hello " * 600

    classification_response = requests.post(
        server.url_for("classify"),
        json={"model": model_name, "input": long_text, "truncate_prompt_tokens": 5},
    )

    classification_response.raise_for_status()
    output = ClassificationResponse.model_validate(classification_response.json())

    assert len(output.data) == 1
    assert output.data[0].index == 0
    assert hasattr(output.data[0], "probs")
    assert output.usage.prompt_tokens == 5
    assert output.usage.total_tokens == 5

    # invalid_truncate_prompt_tokens
    classification_response = requests.post(
        server.url_for("classify"),
        json={"model": model_name, "input": "test", "truncate_prompt_tokens": 513},
    )

    error = classification_response.json()
    assert classification_response.status_code == 400
    assert "truncate_prompt_tokens" in error["error"]["message"]
```
**EN:** This test validates `test_truncate_prompt_tokens`. It uses parameterization over `model_name`. Key inputs are `server`, `model_name`. The main assertion is `len(output.data) == 1` and `output.data[0].index == 0`.
**CN:** 这个测试验证 `test_truncate_prompt_tokens`。 它通过参数化组合 `model_name`。 关键输入包括 `server`、`model_name`。 核心断言是 `len(output.data) == 1` and `output.data[0].index == 0`。

### Test / 测试: test_add_special_tokens (L166-L182)
```python
@pytest.mark.parametrize("model_name", [MODEL_NAME])
def test_add_special_tokens(server: RemoteOpenAIServer, model_name: str):
    # The add_special_tokens parameter doesn't seem to be working with this model.
    # working with papluca/xlm-roberta-base-language-detection
    response = requests.post(
        server.url_for("classify"),
        json={"model": model_name, "input": input_text, "add_special_tokens": False},
    )
    response.raise_for_status()
    ClassificationResponse.model_validate(response.json())

    response = requests.post(
        server.url_for("classify"),
        json={"model": model_name, "input": input_text, "add_special_tokens": True},
    )
    response.raise_for_status()
    ClassificationResponse.model_validate(response.json())
```
**EN:** This test validates `test_add_special_tokens`. It uses parameterization over `model_name`. Key inputs are `server`, `model_name`.
**CN:** 这个测试验证 `test_add_special_tokens`。 它通过参数化组合 `model_name`。 关键输入包括 `server`、`model_name`。

### Test / 测试: test_chat_request (L185-L285)
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

# ... 75 lines omitted for brevity ...
            "continue_final_message": True,
            "add_generation_prompt": True,
        },
    )
    assert (
        "Cannot set both `continue_final_message` and `add_generation_prompt` to True."
        in response.json()["error"]["message"]
    )
```
**EN:** This async test validates `test_chat_request`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `model_name`. The main assertion is `output.object == 'list'` and `output.model == MODEL_NAME`.
**CN:** 这个异步测试验证 `test_chat_request`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`model_name`。 核心断言是 `output.object == 'list'` and `output.model == MODEL_NAME`。

### Test / 测试: test_invocations_completion_request (L288-L315)
```python
@pytest.mark.asyncio
async def test_invocations_completion_request(server: RemoteOpenAIServer):
    request_args = {
        "model": MODEL_NAME,
        "input": input_text,
    }

    classification_response = requests.post(
        server.url_for("classify"), json=request_args
    )
    classification_response.raise_for_status()

    invocation_response = requests.post(
        server.url_for("invocations"), json=request_args
    )
    invocation_response.raise_for_status()

    classification_output = classification_response.json()
    invocation_output = invocation_response.json()

    assert classification_output.keys() == invocation_output.keys()
    for classification_data, invocation_data in zip(
        classification_output["data"], invocation_output["data"]
    ):
        assert classification_data.keys() == invocation_data.keys()
        assert classification_data["probs"] == pytest.approx(
            invocation_data["probs"], rel=0.01
        )
```
**EN:** This async test validates `test_invocations_completion_request`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `classification_output.keys() == invocation_output.keys()` and `classification_data.keys() == invocation_data.keys()`.
**CN:** 这个异步测试验证 `test_invocations_completion_request`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `classification_output.keys() == invocation_output.keys()` and `classification_data.keys() == invocation_data.keys()`。

### Test / 测试: test_invocations_chat_request (L318-L357)
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

    request_args = {"model": MODEL_NAME, "messages": messages}
# ... 14 lines omitted for brevity ...
    assert classification_output.keys() == invocation_output.keys()
    for classification_data, invocation_data in zip(
        classification_output["data"], invocation_output["data"]
    ):
        assert classification_data.keys() == invocation_data.keys()
        assert classification_data["probs"] == pytest.approx(
            invocation_data["probs"], rel=0.01
        )
```
**EN:** This async test validates `test_invocations_chat_request`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `classification_output.keys() == invocation_output.keys()` and `classification_data.keys() == invocation_data.keys()`.
**CN:** 这个异步测试验证 `test_invocations_chat_request`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `classification_output.keys() == invocation_output.keys()` and `classification_data.keys() == invocation_data.keys()`。

### Test / 测试: test_use_activation (L360-L387)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_use_activation(server: RemoteOpenAIServer, model_name: str):
    async def get_outputs(use_activation):
        response = requests.post(
            server.url_for("classify"),
            json={
                "model": model_name,
                "input": input_text,
                "use_activation": use_activation,
            },
        )
        outputs = response.json()
        return torch.tensor([x["probs"] for x in outputs["data"]])

    default = await get_outputs(use_activation=None)
    w_activation = await get_outputs(use_activation=True)
    wo_activation = await get_outputs(use_activation=False)

    assert torch.allclose(default, w_activation, atol=1e-2), (
        "Default should use activation."
    )
    assert not torch.allclose(w_activation, wo_activation, atol=1e-2), (
        "wo_activation should not use activation."
    )
    assert torch.allclose(F.softmax(wo_activation, dim=-1), w_activation, atol=1e-2), (
        "w_activation should be close to activation(wo_activation)."
    )
```
**EN:** This async test validates `test_use_activation`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `model_name`. The main assertion is `torch.allclose(default, w_activation, atol=0.01)` and `not torch.allclose(w_activation, wo_activation, atol=0.01)`.
**CN:** 这个异步测试验证 `test_use_activation`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`model_name`。 核心断言是 `torch.allclose(default, w_activation, atol=0.01)` and `not torch.allclose(w_activation, wo_activation, atol=0.01)`。

### Test / 测试: test_score (L390-L402)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_score(server: RemoteOpenAIServer, model_name: str):
    # Scoring API is only enabled for num_labels == 1.
    response = requests.post(
        server.url_for("score"),
        json={
            "model": model_name,
            "queries": "ping",
            "documents": "pong",
        },
    )
    assert response.json()["detail"] == "Not Found"
```
**EN:** This async test validates `test_score`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `model_name`. The main assertion is `response.json()['detail'] == 'Not Found'`.
**CN:** 这个异步测试验证 `test_score`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`model_name`。 核心断言是 `response.json()['detail'] == 'Not Found'`。

### Test / 测试: test_rerank (L405-L417)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_rerank(server: RemoteOpenAIServer, model_name: str):
    # Scoring API is only enabled for num_labels == 1.
    response = requests.post(
        server.url_for("rerank"),
        json={
            "model": model_name,
            "query": "ping",
            "documents": ["pong"],
        },
    )
    assert response.json()["detail"] == "Not Found"
```
**EN:** This async test validates `test_rerank`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `model_name`. The main assertion is `response.json()['detail'] == 'Not Found'`.
**CN:** 这个异步测试验证 `test_rerank`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`model_name`。 核心断言是 `response.json()['detail'] == 'Not Found'`。

### Test / 测试: test_pooling_classify (L420-L434)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_pooling_classify(server: RemoteOpenAIServer, model_name: str):
    response = requests.post(
        server.url_for("pooling"),
        json={
            "model": model_name,
            "input": input_text,
            "encoding_format": "float",
            "task": "classify",
        },
    )
    poolings = PoolingResponse.model_validate(response.json())
    assert len(poolings.data) == 1
    assert len(poolings.data[0].data) == 2
```
**EN:** This async test validates `test_pooling_classify`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `model_name`. The main assertion is `len(poolings.data) == 1` and `len(poolings.data[0].data) == 2`.
**CN:** 这个异步测试验证 `test_pooling_classify`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`model_name`。 核心断言是 `len(poolings.data) == 1` and `len(poolings.data[0].data) == 2`。

### Test / 测试: test_pooling_not_supported (L437-L460)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("task", ["embed", "token_embed", "token_classify", "plugin"])
async def test_pooling_not_supported(
    server: RemoteOpenAIServer, model_name: str, task: str
):
    response = requests.post(
        server.url_for("pooling"),
        json={
            "model": model_name,
            "input": input_text,
            "encoding_format": "float",
            "task": task,
        },
    )
    assert response.json()["error"]["type"] == "BadRequestError"

    if task == "plugin":
        err_msg = "No IOProcessor plugin installed."
    elif task == "token_classify":
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
- **Third-party / 第三方**: `pytest`, `requests`, `torch`, `torch.nn.functional`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.pooling.classify.protocol.ClassificationResponse`, `vllm.entrypoints.pooling.pooling.protocol.PoolingResponse`
