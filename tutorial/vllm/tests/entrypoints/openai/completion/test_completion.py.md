# test_completion.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/completion/test_completion.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers text completion behavior and OpenAI-compatible serving. The file defines 15 test(s), 3 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖文本补全行为与OpenAI 兼容服务。它定义了 15 个测试、3 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L12)
```python
import openai  # use the official client for correctness check
import pytest
import pytest_asyncio
import regex as re
from openai import BadRequestError

from tests.utils import RemoteOpenAIServer
from vllm.tokenizers import get_tokenizer
```
**EN:** Imports third-party packages like `openai`, `openai.BadRequestError`, `pytest`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.tokenizers.get_tokenizer`.
**CN:** 导入第三方包（如 `openai`、`openai.BadRequestError`、`pytest`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.tokenizers.get_tokenizer`）。

### Module setup / 模块级配置: MODEL_NAME (L15-L15)
```python
MODEL_NAME = "facebook/opt-125m"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: default_server_args (L18-L30)
```python
@pytest.fixture(scope="module")
def default_server_args():
    return [
        "--dtype",
        "float32",
        "--max-model-len",
        "2048",
        "--max-num-seqs",
        "128",
        "--enforce-eager",
        "--enable-prompt-tokens-details",
        "--no-enable-prefix-caching",
    ]
```
**EN:** This fixture prepares `default_server_args` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `default_server_args`。

### Fixture / 夹具: server (L33-L36)
```python
@pytest.fixture(scope="module")
def server(default_server_args):
    with RemoteOpenAIServer(MODEL_NAME, default_server_args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. Key inputs are `default_server_args`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 关键输入包括 `default_server_args`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L39-L42)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_single_completion (L45-L73)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_single_completion(client: openai.AsyncOpenAI, model_name: str) -> None:
    completion = await client.completions.create(
        model=model_name, prompt="Hello, my name is", max_tokens=5, temperature=0.0
    )

    assert completion.id is not None
    assert completion.choices is not None and len(completion.choices) == 1

    choice = completion.choices[0]
    assert len(choice.text) >= 5
    assert choice.finish_reason == "length"
    assert completion.usage == openai.types.CompletionUsage(
        completion_tokens=5, prompt_tokens=6, total_tokens=11
    )

    # test using token IDs
    completion = await client.completions.create(
        model=model_name,
        prompt=[0, 0, 0, 0, 0],
        max_tokens=5,
        temperature=0.0,
    )
    assert len(completion.choices[0].text) >= 1
    assert completion.choices[0].prompt_logprobs is None
```
**EN:** This async test validates `test_single_completion`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `completion.id is not None` and `completion.choices is not None and len(completion.choices) == 1`.
**CN:** 这个异步测试验证 `test_single_completion`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `completion.id is not None` and `completion.choices is not None and len(completion.choices) == 1`。

### Test / 测试: test_no_logprobs (L76-L91)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_no_logprobs(client: openai.AsyncOpenAI, model_name: str):
    # test using token IDs
    completion = await client.completions.create(
        model=model_name,
        prompt=[0, 0, 0, 0, 0],
        max_tokens=5,
        temperature=0.0,
        logprobs=None,
    )
    choice = completion.choices[0]
    assert choice.logprobs is None
```
**EN:** This async test validates `test_no_logprobs`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `choice.logprobs is None`.
**CN:** 这个异步测试验证 `test_no_logprobs`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `choice.logprobs is None`。

### Test / 测试: test_zero_logprobs (L94-L112)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_zero_logprobs(client: openai.AsyncOpenAI, model_name: str):
    # test using token IDs
    completion = await client.completions.create(
        model=model_name,
        prompt=[0, 0, 0, 0, 0],
        max_tokens=5,
        temperature=0.0,
        logprobs=0,
    )
    choice = completion.choices[0]
    assert choice.logprobs is not None
    assert choice.logprobs.token_logprobs is not None
    assert choice.logprobs.top_logprobs is not None
    assert len(choice.logprobs.top_logprobs[0]) == 1
```
**EN:** This async test validates `test_zero_logprobs`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `choice.logprobs is not None` and `choice.logprobs.token_logprobs is not None`.
**CN:** 这个异步测试验证 `test_zero_logprobs`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `choice.logprobs is not None` and `choice.logprobs.token_logprobs is not None`。

### Test / 测试: test_some_logprobs (L115-L133)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_some_logprobs(client: openai.AsyncOpenAI, model_name: str):
    # test using token IDs
    completion = await client.completions.create(
        model=model_name,
        prompt=[0, 0, 0, 0, 0],
        max_tokens=5,
        temperature=0.0,
        logprobs=5,
    )
    choice = completion.choices[0]
    assert choice.logprobs is not None
    assert choice.logprobs.token_logprobs is not None
    assert choice.logprobs.top_logprobs is not None
    assert 5 <= len(choice.logprobs.top_logprobs[0]) <= 6
```
**EN:** This async test validates `test_some_logprobs`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `choice.logprobs is not None` and `choice.logprobs.token_logprobs is not None`.
**CN:** 这个异步测试验证 `test_some_logprobs`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `choice.logprobs is not None` and `choice.logprobs.token_logprobs is not None`。

### Test / 测试: test_too_many_completion_logprobs (L136-L180)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_too_many_completion_logprobs(
    client: openai.AsyncOpenAI, model_name: str
) -> None:
    with pytest.raises(
        (openai.BadRequestError, openai.APIError)
    ):  # test using token IDs
        await client.completions.create(
            model=model_name,
            prompt=[0, 0, 0, 0, 0],
            max_tokens=5,
            temperature=0.0,
            # vLLM has higher default max_logprobs (20 instead of 5) to support
            # both Completion API and Chat Completion API
# ... 19 lines omitted for brevity ...
    # the server should still work afterwards
    completion = await client.completions.create(
        model=model_name,
        prompt=[0, 0, 0, 0, 0],
        max_tokens=5,
        temperature=0.0,
    )
    assert len(completion.choices[0].text) >= 0
```
**EN:** This async test validates `test_too_many_completion_logprobs`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `len(completion.choices[0].text) >= 0`.
**CN:** 这个异步测试验证 `test_too_many_completion_logprobs`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(completion.choices[0].text) >= 0`。

### Test / 测试: test_prompt_logprobs_completion (L183-L211)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name, prompt_logprobs",
    [(MODEL_NAME, -1), (MODEL_NAME, 0), (MODEL_NAME, 1), (MODEL_NAME, None)],
)
async def test_prompt_logprobs_completion(
    client: openai.AsyncOpenAI, model_name: str, prompt_logprobs: int | None
):
    params: dict = {
        "prompt": ["A robot may not injure another robot", "My name is"],
        "model": model_name,
    }
    if prompt_logprobs is not None:
        params["extra_body"] = {"prompt_logprobs": prompt_logprobs}

    if prompt_logprobs is not None and prompt_logprobs < 0:
        with pytest.raises(BadRequestError):
            await client.completions.create(**params)
    else:
        completion = await client.completions.create(**params)
        if prompt_logprobs is not None:
            assert completion.choices[0].prompt_logprobs is not None
            assert len(completion.choices[0].prompt_logprobs) > 0

            assert completion.choices[1].prompt_logprobs is not None
            assert len(completion.choices[1].prompt_logprobs) > 0

        else:
            assert completion.choices[0].prompt_logprobs is None
```
**EN:** This async test validates `test_prompt_logprobs_completion`. It uses parameterization over `model_name`, `prompt_logprobs`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `prompt_logprobs`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `completion.choices[0].prompt_logprobs is not None` and `len(completion.choices[0].prompt_logprobs) > 0`.
**CN:** 这个异步测试验证 `test_prompt_logprobs_completion`。 它通过参数化组合 `model_name`、`prompt_logprobs`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`prompt_logprobs`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `completion.choices[0].prompt_logprobs is not None` and `len(completion.choices[0].prompt_logprobs) > 0`。

### Test / 测试: test_completion_streaming (L214-L244)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_completion_streaming(
    client: openai.AsyncOpenAI, model_name: str
) -> None:
    prompt = "What is an LLM?"

    single_completion = await client.completions.create(
        model=model_name,
        prompt=prompt,
        max_tokens=5,
        temperature=0.0,
    )
    single_output = single_completion.choices[0].text
    stream = await client.completions.create(
# ... 5 lines omitted for brevity ...
        chunks.append(chunk.choices[0].text)
        if chunk.choices[0].finish_reason is not None:
            finish_reason_count += 1
    # finish reason should only return in last block
    assert finish_reason_count == 1
    assert chunk.choices[0].finish_reason == "length"
    assert chunk.choices[0].text
    assert "".join(chunks) == single_output
```
**EN:** This async test validates `test_completion_streaming`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `finish_reason_count == 1` and `chunk.choices[0].finish_reason == 'length'`.
**CN:** 这个异步测试验证 `test_completion_streaming`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `finish_reason_count == 1` and `chunk.choices[0].finish_reason == 'length'`。

### Test / 测试: test_parallel_no_streaming (L247-L293)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_parallel_no_streaming(client: openai.AsyncOpenAI, model_name: str):
    """Parallel sampling without streaming.
    A single request output contains a list of completions.
    """

    prompt = "What is an LLM?"
    n = 3
    max_tokens = 50  # we want some to finish earlier than others

    # High temperature to maximize chance of unique completions.
    completion = await client.completions.create(
        model=model_name,
        prompt=prompt,
# ... 21 lines omitted for brevity ...
    assert len(output_token_lengths) > 1
    # Assert `n` unique completions
    num_unique = len(completion_repeats)
    if num_unique != n:
        repeats = {txt: num for (txt, num) in completion_repeats.items() if num > 1}
        raise AssertionError(
            f"Expected {n} unique completions, got {num_unique}; repeats: {repeats}."
        )
```
**EN:** This async test validates `test_parallel_no_streaming`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `num_completions == n` and `len(output_token_lengths) > 1`.
**CN:** 这个异步测试验证 `test_parallel_no_streaming`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `num_completions == n` and `len(output_token_lengths) > 1`。

### Test / 测试: test_parallel_streaming (L296-L352)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_parallel_streaming(client: openai.AsyncOpenAI, model_name: str):
    """Streaming for parallel sampling.
    The tokens from multiple samples, are flattened into a single stream,
    with an index to indicate which sample the token belongs to.
    """

    prompt = "What is an LLM?"
    n = 3
    max_tokens = 50  # we want some to finish earlier than others

    stream = await client.completions.create(
        model=model_name,
        prompt=prompt,
# ... 31 lines omitted for brevity ...
    assert len(chunk_lengths) > 1
    # Assert `n` unique completions
    num_unique = len(completion_repeats)
    if num_unique != n:
        repeats = {txt: num for (txt, num) in completion_repeats.items() if num > 1}
        raise AssertionError(
            f"{num_unique} unique completions, expected {n}; repeats: {repeats}"
        )
```
**EN:** This async test validates `test_parallel_streaming`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `finish_reason_count == n` and `len(chunk_lengths) > 1`.
**CN:** 这个异步测试验证 `test_parallel_streaming`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `finish_reason_count == n` and `len(chunk_lengths) > 1`。

### Test / 测试: test_completion_stream_options (L355-L511)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_completion_stream_options(client: openai.AsyncOpenAI, model_name: str):
    prompt = "What is the capital of France?"

    # Test stream=True, stream_options=
    #     {"include_usage": False, "continuous_usage_stats": False}
    stream = await client.completions.create(
        model=model_name,
        prompt=prompt,
        max_tokens=5,
        temperature=0.0,
        stream=True,
        stream_options={
            "include_usage": False,
# ... 131 lines omitted for brevity ...
        await client.completions.create(
            model=model_name,
            prompt=prompt,
            max_tokens=5,
            temperature=0.0,
            stream=False,
            stream_options={"continuous_usage_stats": True},
        )
```
**EN:** This async test validates `test_completion_stream_options`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `chunk.usage is None` and `chunk.usage is None`.
**CN:** 这个异步测试验证 `test_completion_stream_options`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `chunk.usage is None` and `chunk.usage is None`。

### Test / 测试: test_batch_completions (L514-L569)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_batch_completions(client: openai.AsyncOpenAI, model_name: str):
    # test both text and token IDs
    for prompts in (["Hello, my name is"] * 2, [[0, 0, 0, 0, 0]] * 2):
        # test simple list
        batch = await client.completions.create(
            model=model_name,
            prompt=prompts,
            max_tokens=5,
            temperature=0.0,
        )
        assert len(batch.choices) == 2
        assert batch.choices[0].text == batch.choices[1].text

# ... 30 lines omitted for brevity ...
            stream=True,
        )
        texts = [""] * 2
        async for chunk in batch:
            assert len(chunk.choices) == 1
            choice = chunk.choices[0]
            texts[choice.index] += choice.text
        assert texts[0] == texts[1]
```
**EN:** This async test validates `test_batch_completions`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `len(batch.choices) == 2` and `batch.choices[0].text == batch.choices[1].text`.
**CN:** 这个异步测试验证 `test_batch_completions`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(batch.choices) == 2` and `batch.choices[0].text == batch.choices[1].text`。

### Test / 测试: test_echo_logprob_completion (L572-L602)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
@pytest.mark.parametrize("logprobs_arg", [1, 0])
async def test_echo_logprob_completion(
    client: openai.AsyncOpenAI, model_name: str, logprobs_arg: int
):
    tokenizer = get_tokenizer(tokenizer_name=MODEL_NAME)
    # test using text and token IDs
    for prompt in ("Hello, my name is", [0, 0, 0, 0, 0]):
        completion = await client.completions.create(
            model=model_name,
            prompt=prompt,
            max_tokens=5,
            temperature=0.0,
            echo=True,
# ... 5 lines omitted for brevity ...
        logprobs = completion.choices[0].logprobs
        assert logprobs is not None
        assert len(logprobs.text_offset) > 5
        assert len(logprobs.token_logprobs) > 5 and logprobs.token_logprobs[0] is None
        assert len(logprobs.top_logprobs) > 5 and logprobs.top_logprobs[0] is None
        for top_logprobs in logprobs.top_logprobs[1:]:
            assert max(logprobs_arg, 1) <= len(top_logprobs) <= logprobs_arg + 1
        assert len(logprobs.tokens) > 5
```
**EN:** This async test validates `test_echo_logprob_completion`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `logprobs_arg`. It drives client-facing request creation through the API surface under test. The main assertion is `re.search('^' + prompt_text, completion.choices[0].text)` and `logprobs is not None`.
**CN:** 这个异步测试验证 `test_echo_logprob_completion`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`logprobs_arg`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `re.search('^' + prompt_text, completion.choices[0].text)` and `logprobs is not None`。

### Test / 测试: test_invalid_json_schema (L605-L638)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_invalid_json_schema(client: openai.AsyncOpenAI, model_name: str) -> None:
    invalid_json_schema = {
        "$defs": {
            "CarType": {
                "enum": ["sedan", "SUV", "Truck", "Coupe"],
                "title": "CarType",
                "type": "string",
            }
        },
        "properties": {
            "brand": {"title": "Brand", "type": "string"},
            "model": {"title": "Model", "type": "string"},
            "car_type": {"$ref": "#/$defs/CarType"},
# ... 8 lines omitted for brevity ...
        "the most iconic car from the 90's"
    )
    with pytest.raises((openai.BadRequestError, openai.APIError)):
        await client.completions.create(
            model=model_name,
            prompt=prompt,
            extra_body={"structured_outputs": {"json": invalid_json_schema}},
        )
```
**EN:** This async test validates `test_invalid_json_schema`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_invalid_json_schema`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_invalid_regex (L641-L658)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_invalid_regex(client: openai.AsyncOpenAI, model_name: str):
    prompt = (
        "Generate an email address for Alan Turing, who works in Enigma."
        "End in .com and new line. Example result:"
        "alan.turing@enigma.com\n"
    )

    with pytest.raises((openai.BadRequestError, openai.APIError)):
        await client.completions.create(
            model=model_name,
            prompt=prompt,
            extra_body={"structured_outputs": {"regex": r"[.*"}, "stop": ["\n"]},
        )
```
**EN:** This async test validates `test_invalid_regex`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_invalid_regex`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_invalid_grammar (L661-L692)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_invalid_grammar(client: openai.AsyncOpenAI, model_name: str):
    invalid_simplified_sql_grammar = """
        root ::= select_statementinvalidsyntax

        select_statement ::= "SELECT " column " from " table " where " condition

        column ::= "col_1 " | "col_2 "

        table ::= "table_1 " | "table_2 "

        condition ::= column "= " number

        number ::= "1 " | "2 "
# ... 6 lines omitted for brevity ...
    with pytest.raises((openai.BadRequestError, openai.APIError)):
        await client.completions.create(
            model=model_name,
            prompt=prompt,
            extra_body={
                "structured_outputs": {"grammar": invalid_simplified_sql_grammar}
            },
        )
```
**EN:** This async test validates `test_invalid_grammar`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_invalid_grammar`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

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
- **Third-party / 第三方**: `openai`, `openai.BadRequestError`, `pytest`, `pytest_asyncio`, `regex`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.tokenizers.get_tokenizer`
