# test_online_dimensions.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/embed/test_online_dimensions.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior. The file defines 1 test(s), 4 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为。它定义了 1 个测试、4 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L7-L15)
```python
import openai
import pytest

from tests.conftest import HfRunner
from tests.models.language.pooling.embed_utils import run_embedding_correctness_test
from tests.models.utils import EmbedModelInfo
from tests.utils import ROCM_EXTRA_ARGS, RemoteOpenAIServer
from vllm.entrypoints.pooling.embed.protocol import EmbeddingResponse
from vllm.platforms import current_platform
```
**EN:** Imports third-party packages like `openai`, `pytest`, project helpers such as `tests.conftest.HfRunner`, `tests.models.language.pooling.embed_utils.run_embedding_correctness_test`, `tests.models.utils.EmbedModelInfo`.
**CN:** 导入第三方包（如 `openai`、`pytest`）、项目内辅助模块（如 `tests.conftest.HfRunner`、`tests.models.language.pooling.embed_utils.run_embedding_correctness_test`、`tests.models.utils.EmbedModelInfo`）。

### Module setup / 模块级配置: MODELS, input_texts (L17-L28)
```python
MODELS = [
    EmbedModelInfo("intfloat/multilingual-e5-small", is_matryoshka=False),
    EmbedModelInfo(
        "Snowflake/snowflake-arctic-embed-m-v1.5",
        is_matryoshka=True,
        matryoshka_dimensions=[256],
    ),
]

input_texts = [
    "The chef prepared a delicious meal.",
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODELS`, `input_texts`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODELS`、`input_texts`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: model_info (L31-L33)
```python
@pytest.fixture(scope="module", params=MODELS)
def model_info(request):
    return request.param
```
**EN:** This fixture prepares `model_info` for dependent tests. Key inputs are `request`.
**CN:** 这个 fixture 为依赖它的测试准备 `model_info`。 关键输入包括 `request`。

### Fixture / 夹具: dtype (L36-L38)
```python
@pytest.fixture(scope="module", params=["bfloat16"])
def dtype(request):
    return request.param
```
**EN:** This fixture prepares `dtype` for dependent tests. Key inputs are `request`.
**CN:** 这个 fixture 为依赖它的测试准备 `dtype`。 关键输入包括 `request`。

### Fixture / 夹具: server (L41-L65)
```python
@pytest.fixture(scope="module")
def server(model_info, dtype: str):
    args = [
        "--runner",
        "pooling",
        # use half precision for speed and memory savings in CI environment
        "--dtype",
        dtype,
        "--enforce-eager",
        "--max-model-len",
        "512",
    ] + ROCM_EXTRA_ARGS

    if model_info.name == "Snowflake/snowflake-arctic-embed-m-v1.5":
        # Manually enable Matryoshka Embeddings
        args.extend(
            ["--trust_remote_code", "--hf_overrides", '{"matryoshka_dimensions":[256]}']
        )

    # ROCm: Use Flex Attention to support encoder-only self-attention.
    if current_platform.is_rocm():
        args.extend(["--attention-backend", "FLEX_ATTENTION"])

    with RemoteOpenAIServer(model_info.name, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. Key inputs are `model_info`, `dtype`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 关键输入包括 `model_info`、`dtype`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: hf_model (L68-L73)
```python
@pytest.fixture(scope="module")
def hf_model(hf_runner, model_info, dtype: str):
    with hf_runner(
        model_info.name, dtype=dtype, is_sentence_transformer=True
    ) as hf_model:
        yield hf_model
```
**EN:** This fixture prepares `hf_model` for dependent tests. Key inputs are `hf_runner`, `model_info`, `dtype`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `hf_model`。 关键输入包括 `hf_runner`、`model_info`、`dtype`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_matryoshka (L76-L131)
```python
@pytest.mark.asyncio
async def test_matryoshka(
    model_info: EmbedModelInfo, server: RemoteOpenAIServer, hf_model: HfRunner
):
    client = server.get_async_client()

    async def make_request_and_correctness_test(dimensions):
        prompts = input_texts * 3

        embedding_response = await client.embeddings.create(
            model=model_info.name,
            input=prompts,
            dimensions=dimensions,
            encoding_format="float",
        )
        embeddings = EmbeddingResponse.model_validate(
            embedding_response.model_dump(mode="json")
        )
# ... 30 lines omitted for brevity ...

    else:
        for dimensions in [None]:
            await make_request_and_correctness_test(dimensions)

        for dimensions in [-1, 16]:
            with pytest.raises(openai.BadRequestError):
                await make_request_and_correctness_test(dimensions)
```
**EN:** This async test validates `test_matryoshka`. Relevant pytest markers include `asyncio`. Key inputs are `model_info`, `server`, `hf_model`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `embeddings.id is not None` and `len(embeddings.data) == 3`.
**CN:** 这个异步测试验证 `test_matryoshka`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `model_info`、`server`、`hf_model`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `embeddings.id is not None` and `len(embeddings.data) == 3`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `openai`, `pytest`
- **Project / 项目内**: `tests.conftest.HfRunner`, `tests.models.language.pooling.embed_utils.run_embedding_correctness_test`, `tests.models.utils.EmbedModelInfo`, `tests.utils.ROCM_EXTRA_ARGS`, `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.pooling.embed.protocol.EmbeddingResponse`, `vllm.platforms.current_platform`
