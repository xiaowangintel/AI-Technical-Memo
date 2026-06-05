# test_completion_with_prompt_embeds.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/completion/test_completion_with_prompt_embeds.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers text completion behavior and OpenAI-compatible serving. The file defines 5 test(s), 4 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖文本补全行为与OpenAI 兼容服务。它定义了 5 个测试、4 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L17)
```python
import io
import json

import openai  # use the official client for correctness check
import pybase64 as base64
import pytest
import pytest_asyncio
import torch

# downloading lora to test lora requests
from openai import BadRequestError
from transformers import AutoConfig

from tests.utils import RemoteOpenAIServer
```
**EN:** Imports standard-library modules such as `io`, `json`, third-party packages like `openai`, `openai.BadRequestError`, `pybase64`, project helpers such as `tests.utils.RemoteOpenAIServer`.
**CN:** 导入标准库模块（如 `io`、`json`）、第三方包（如 `openai`、`openai.BadRequestError`、`pybase64`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: MODEL_NAME, LORA_SERVING_MODEL_NAME, CONFIG (L20-L23)
```python
MODEL_NAME = "facebook/opt-125m"
LORA_SERVING_MODEL_NAME = "opt125m-lora"

CONFIG = AutoConfig.from_pretrained(MODEL_NAME)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `LORA_SERVING_MODEL_NAME`, `CONFIG`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`LORA_SERVING_MODEL_NAME`、`CONFIG`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: default_server_args (L26-L62)
```python
@pytest.fixture(scope="module", params=["use-lora"])
def default_server_args(
    request: pytest.FixtureRequest, opt125_lora_files: str
) -> list[str]:
    args = [
        # use half precision for speed and memory savings in CI environment
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "2048",
        "--max-num-seqs",
        "128",
        "--enforce-eager",
        # Prompt Embeds server args
        "--enable-prompt-embeds",
    ]

    if request.param == "use-lora":
# ... 11 lines omitted for brevity ...
                "--max-lora-rank",
                "64",
                "--max-cpu-loras",
                "2",
            ]
        )

    return args
```
**EN:** This fixture prepares `default_server_args` for dependent tests. Key inputs are `request`, `opt125_lora_files`.
**CN:** 这个 fixture 为依赖它的测试准备 `default_server_args`。 关键输入包括 `request`、`opt125_lora_files`。

### Module setup / 模块级配置: EXAMPLE_PROMPTS (L65-L68)
```python
EXAMPLE_PROMPTS = [
    "Hello, my name is",
    "What is an LLM?",
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `EXAMPLE_PROMPTS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `EXAMPLE_PROMPTS`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _encode_embeds (L71-L74)
```python
def _encode_embeds(embeds: torch.Tensor):
    buffer = io.BytesIO()
    torch.save(embeds, buffer)
    return base64.b64encode(buffer.getvalue()).decode("utf-8")
```
**EN:** This helper encapsulates reusable logic in `_encode_embeds`. Key inputs are `embeds`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_encode_embeds` 中。 关键输入包括 `embeds`。 它把计算得到的状态或辅助对象返回给调用方。

### Fixture / 夹具: example_prompt_embeds (L77-L83)
```python
@pytest.fixture(scope="module")
def example_prompt_embeds(hf_runner):
    """Create example embeddings and return them as base64 encoded string."""
    with hf_runner(MODEL_NAME) as hf_model:
        example_embeddings = hf_model.get_prompt_embeddings(EXAMPLE_PROMPTS)

    return [_encode_embeds(item) for item in example_embeddings]
```
**EN:** This fixture prepares `example_prompt_embeds` for dependent tests. Key inputs are `hf_runner`.
**CN:** 这个 fixture 为依赖它的测试准备 `example_prompt_embeds`。 关键输入包括 `hf_runner`。

### Fixture / 夹具: server_with_prompt_embeds (L86-L89)
```python
@pytest.fixture(scope="module")
def server_with_prompt_embeds(default_server_args):
    with RemoteOpenAIServer(MODEL_NAME, default_server_args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server_with_prompt_embeds` for dependent tests. Key inputs are `default_server_args`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server_with_prompt_embeds`。 关键输入包括 `default_server_args`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client_with_prompt_embeds (L92-L95)
```python
@pytest_asyncio.fixture
async def client_with_prompt_embeds(server_with_prompt_embeds):
    async with server_with_prompt_embeds.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client_with_prompt_embeds` for dependent tests. Key inputs are `server_with_prompt_embeds`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client_with_prompt_embeds`。 关键输入包括 `server_with_prompt_embeds`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_completions_with_prompt_embeds (L98-L204)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME, LORA_SERVING_MODEL_NAME])
async def test_completions_with_prompt_embeds(
    example_prompt_embeds,
    client_with_prompt_embeds: openai.AsyncOpenAI,
    model_name: str,
):
    encoded_embeds, encoded_embeds2 = example_prompt_embeds

    # Test case: Single prompt embeds input
    completion = await client_with_prompt_embeds.completions.create(
        model=model_name,
        prompt=None,
        max_tokens=5,
        temperature=0.0,
        extra_body={"prompt_embeds": encoded_embeds},
    )
    assert len(completion.choices[0].text) >= 1
# ... 81 lines omitted for brevity ...
        prompt=None,
        max_tokens=5,
        temperature=0.0,
        extra_body={"prompt_embeds": encoded_embeds},
    )
    # Embeddings responses should be handled first
    assert completion_mixed.choices[0].text == completion_embeds_only.choices[0].text
    assert completion_mixed.choices[1].text == completion_text_only.choices[0].text
```
**EN:** This async test validates `test_completions_with_prompt_embeds`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `example_prompt_embeds`, `client_with_prompt_embeds`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `len(completion.choices[0].text) >= 1` and `completion.choices[0].prompt_logprobs is None`.
**CN:** 这个异步测试验证 `test_completions_with_prompt_embeds`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `example_prompt_embeds`、`client_with_prompt_embeds`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(completion.choices[0].text) >= 1` and `completion.choices[0].prompt_logprobs is None`。

### Test / 测试: test_completions_errors_with_prompt_embeds (L207-L220)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME, LORA_SERVING_MODEL_NAME])
async def test_completions_errors_with_prompt_embeds(
    client_with_prompt_embeds: openai.AsyncOpenAI, model_name: str
):
    # Test error case: invalid prompt_embeds
    with pytest.raises(BadRequestError):
        await client_with_prompt_embeds.completions.create(
            prompt=None,
            model=model_name,
            max_tokens=5,
            temperature=0.0,
            extra_body={"prompt_embeds": "invalid_base64"},
        )
```
**EN:** This async test validates `test_completions_errors_with_prompt_embeds`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client_with_prompt_embeds`, `model_name`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_completions_errors_with_prompt_embeds`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client_with_prompt_embeds`、`model_name`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_completions_with_logprobs_and_prompt_embeds (L223-L274)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("logprobs_arg", [1, 0])
@pytest.mark.parametrize("model_name", [MODEL_NAME, LORA_SERVING_MODEL_NAME])
async def test_completions_with_logprobs_and_prompt_embeds(
    example_prompt_embeds,
    client_with_prompt_embeds: openai.AsyncOpenAI,
    logprobs_arg: int,
    model_name: str,
):
    encoded_embeds, encoded_embeds2 = example_prompt_embeds

    # Test case: Logprobs using prompt_embeds
    completion = await client_with_prompt_embeds.completions.create(
        model=model_name,
        prompt=None,
        max_tokens=5,
        temperature=0.0,
        echo=False,
# ... 26 lines omitted for brevity ...
        logprobs = choice.logprobs
        assert logprobs is not None
        assert len(logprobs.text_offset) == 5
        assert len(logprobs.token_logprobs) == 5
        assert len(logprobs.top_logprobs) == 5
        for top_logprobs in logprobs.top_logprobs[1:]:
            assert max(logprobs_arg, 1) <= len(top_logprobs) <= logprobs_arg + 1
        assert len(logprobs.tokens) == 5
```
**EN:** This async test validates `test_completions_with_logprobs_and_prompt_embeds`. It uses parameterization over `logprobs_arg`. Relevant pytest markers include `asyncio`. Key inputs are `example_prompt_embeds`, `client_with_prompt_embeds`, `logprobs_arg`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `logprobs is not None` and `len(logprobs.text_offset) == 5`.
**CN:** 这个异步测试验证 `test_completions_with_logprobs_and_prompt_embeds`。 它通过参数化组合 `logprobs_arg`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `example_prompt_embeds`、`client_with_prompt_embeds`、`logprobs_arg`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `logprobs is not None` and `len(logprobs.text_offset) == 5`。

### Test / 测试: test_prompt_logprobs_raises_error (L277-L291)
```python
@pytest.mark.asyncio
async def test_prompt_logprobs_raises_error(
    example_prompt_embeds,
    client_with_prompt_embeds: openai.AsyncOpenAI,
):
    encoded_embeds, _ = example_prompt_embeds

    with pytest.raises(BadRequestError, match="not compatible"):
        await client_with_prompt_embeds.completions.create(
            model=MODEL_NAME,
            prompt=None,
            max_tokens=5,
            temperature=0.0,
            extra_body={"prompt_embeds": encoded_embeds, "prompt_logprobs": True},
        )
```
**EN:** This async test validates `test_prompt_logprobs_raises_error`. Relevant pytest markers include `asyncio`. Key inputs are `example_prompt_embeds`, `client_with_prompt_embeds`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_prompt_logprobs_raises_error`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `example_prompt_embeds`、`client_with_prompt_embeds`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_empty_prompt_embeds (L294-L304)
```python
@pytest.mark.asyncio
async def test_empty_prompt_embeds(
    client_with_prompt_embeds: openai.AsyncOpenAI,
) -> None:
    await client_with_prompt_embeds.completions.create(
        model=MODEL_NAME,
        prompt="Hello",
        max_tokens=5,
        temperature=0.0,
        extra_body={"prompt_embeds": []},
    )
```
**EN:** This async test validates `test_empty_prompt_embeds`. Relevant pytest markers include `asyncio`. Key inputs are `client_with_prompt_embeds`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_empty_prompt_embeds`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client_with_prompt_embeds`。 它通过被测 API 表面触发面向客户端的请求创建流程。

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
- **Stdlib / 标准库**: `io`, `json`
- **Third-party / 第三方**: `openai`, `openai.BadRequestError`, `pybase64`, `pytest`, `pytest_asyncio`, `torch`, `transformers.AutoConfig`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`
