# test_online.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/token_embed/test_online.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior. The file defines 2 test(s), 1 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为。它定义了 2 个测试、1 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L9)
```python
import pytest
import requests

from tests.utils import RemoteOpenAIServer
from vllm.entrypoints.pooling.pooling.protocol import PoolingResponse
```
**EN:** Imports third-party packages like `pytest`, `requests`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.pooling.pooling.protocol.PoolingResponse`.
**CN:** 导入第三方包（如 `pytest`、`requests`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.entrypoints.pooling.pooling.protocol.PoolingResponse`）。

### Module setup / 模块级配置: MODEL_NAME, DTYPE, input_text (L11-L32)
```python
MODEL_NAME = "intfloat/multilingual-e5-small"
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
    8060,
    7,
    5941,
    12921,
    115774,
    2,
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `DTYPE`, `input_text`, `input_tokens`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`DTYPE`、`input_text`、`input_tokens`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L35-L50)
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
        "--pooler-config.task",
        "token_embed",
    ]

    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_pooling_token_embed (L53-L71)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_pooling_token_embed(server: RemoteOpenAIServer, model_name: str):
    task = "token_embed"
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
    assert len(poolings.data[0].data) == len(input_tokens)
    assert len(poolings.data[0].data[0]) == 384
```
**EN:** This async test validates `test_pooling_token_embed`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `model_name`. The main assertion is `len(poolings.data) == 1` and `len(poolings.data[0].data) == len(input_tokens)`.
**CN:** 这个异步测试验证 `test_pooling_token_embed`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`model_name`。 核心断言是 `len(poolings.data) == 1` and `len(poolings.data[0].data) == len(input_tokens)`。

### Test / 测试: test_pooling_not_supported (L74-L97)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("task", ["embed", "classify", "token_classify", "plugin"])
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
    elif task == "embed":
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
- **Third-party / 第三方**: `pytest`, `requests`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.pooling.pooling.protocol.PoolingResponse`
