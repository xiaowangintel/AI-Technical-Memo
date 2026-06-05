# test_serving_tokens.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/serve/disagg/test_serving_tokens.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers serve subsystem behavior. The file defines 7 test(s), 4 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖服务子系统行为。它定义了 7 个测试、4 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L15)
```python
import json
import os

import httpx
import pytest
import pytest_asyncio
from transformers import AutoTokenizer

from tests.utils import RemoteOpenAIServer
from vllm.config import ModelConfig
from vllm.config.utils import getattr_iter
from vllm.v1.engine.detokenizer import check_stop_strings
```
**EN:** Imports standard-library modules such as `json`, `os`, third-party packages like `httpx`, `pytest`, `pytest_asyncio`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.config.ModelConfig`, `vllm.config.utils.getattr_iter`.
**CN:** 导入标准库模块（如 `json`、`os`）、第三方包（如 `httpx`、`pytest`、`pytest_asyncio`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.config.ModelConfig`、`vllm.config.utils.getattr_iter`）。

### Module setup / 模块级配置: MODEL_NAME, GEN_ENDPOINT (L17-L18)
```python
MODEL_NAME = "Qwen/Qwen3-0.6B"
GEN_ENDPOINT = "/inference/v1/generate"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `GEN_ENDPOINT`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`GEN_ENDPOINT`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: get_vocab_size (L21-L27)
```python
def get_vocab_size(model_name):
    config = ModelConfig(
        model=model_name,
        seed=0,
        dtype="bfloat16",
    )
    return config.get_vocab_size()
```
**EN:** This helper encapsulates reusable logic in `get_vocab_size`. Key inputs are `model_name`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_vocab_size` 中。 关键输入包括 `model_name`。 它把计算得到的状态或辅助对象返回给调用方。

### Fixture / 夹具: tokenizer (L30-L32)
```python
@pytest.fixture(scope="module")
def tokenizer():
    return AutoTokenizer.from_pretrained(MODEL_NAME)
```
**EN:** This fixture prepares `tokenizer` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `tokenizer`。

### Fixture / 夹具: messages (L35-L40)
```python
@pytest.fixture(scope="module")
def messages():
    return [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "How many countries are in the EU?"},
    ]
```
**EN:** This fixture prepares `messages` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `messages`。

### Fixture / 夹具: server (L43-L87)
```python
@pytest.fixture(scope="module")
def server(request):
    args = [
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "1024",
        "--enforce-eager",
        # On ROCm (e.g. MI355X/gfx950), bf16 GEMM results can differ by
        # 1 ULP when the batch dimension (M) changes, because different M
        # values cause the Tensile backend to select different tile
        # configurations with different fp32 accumulation orders. With
        # prefix caching, cache-miss prefills compute all tokens in one
        # pass (large M) while cache-hit requests compute only the
        # uncached suffix (small M), seeding a divergence that amplifies
        # through the residual stream and flips argmax tokens.
        # See: https://github.com/vllm-project/vllm/issues/33123
        #
# ... 19 lines omitted for brevity ...
        )

    envs = os.environ.copy()
    # See: https://github.com/vllm-project/vllm/pull/33493#issuecomment-3888060787
    envs["VLLM_ROCM_USE_SKINNY_GEMM"] = "0"

    with RemoteOpenAIServer(MODEL_NAME, args, env_dict=envs) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. Key inputs are `request`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 关键输入包括 `request`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L90-L100)
```python
@pytest_asyncio.fixture
async def client(server: RemoteOpenAIServer):
    transport = httpx.AsyncHTTPTransport(uds=server.uds) if server.uds else None
    headers = {"Authorization": f"Bearer {server.DUMMY_API_KEY}"}
    async with httpx.AsyncClient(
        transport=transport,
        base_url=server.url_root,
        timeout=600,
        headers=headers,
    ) as c:
        yield c
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_generate_endpoint (L103-L114)
```python
@pytest.mark.asyncio
async def test_generate_endpoint(client):
    payload = {
        "model": MODEL_NAME,
        "token_ids": [1, 2, 3],
        "sampling_params": {"max_tokens": 5},
        "stream": False,
    }
    resp = await client.post(GEN_ENDPOINT, json=payload)
    resp.raise_for_status()
    data = resp.json()
    assert "choices" in data
```
**EN:** This async test validates `test_generate_endpoint`. Relevant pytest markers include `asyncio`. Key inputs are `client`. The main assertion is `'choices' in data`.
**CN:** 这个异步测试验证 `test_generate_endpoint`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 核心断言是 `'choices' in data`。

### Test / 测试: test_generate_defaults_max_tokens_when_omitted (L117-L144)
```python
@pytest.mark.asyncio
async def test_generate_defaults_max_tokens_when_omitted(client):
    """Regression: omitting ``max_tokens`` must not silently cap at 16.

    ``SamplingParams.max_tokens`` defaults to 16. Before the server-side
    defaulting was wired up, every request that didn't set ``max_tokens``
    truncated mid-generation. The server should now fill it in from
    ``max_model_len - prompt_len`` (matching ``/v1/chat/completions``).
    """
    payload = {
        "model": MODEL_NAME,
        "token_ids": [1, 2, 3],
        "sampling_params": {
            "temperature": 0.0,
            "ignore_eos": True,
        },
        "stream": False,
    }
    resp = await client.post(GEN_ENDPOINT, json=payload)
    resp.raise_for_status()
    data = resp.json()
    completion_tokens = len(data["choices"][0]["token_ids"])
    # max_model_len=1024 in the test fixture, prompt is 3 tokens, so we
    # should get ~1021 tokens of output (capped at max_model_len boundary).
    assert completion_tokens > 16, (
        f"expected server-side default to exceed the legacy 16-token cap, "
        f"got {completion_tokens}"
    )
```
**EN:** This async test validates `test_generate_defaults_max_tokens_when_omitted`. Relevant pytest markers include `asyncio`. Key inputs are `client`. The main assertion is `completion_tokens > 16`.
**CN:** 这个异步测试验证 `test_generate_defaults_max_tokens_when_omitted`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 核心断言是 `completion_tokens > 16`。

### Test / 测试: test_generate_stream (L147-L192)
```python
@pytest.mark.asyncio
async def test_generate_stream(client):
    payload = {
        "model": MODEL_NAME,
        "token_ids": [1, 2, 3],
        "sampling_params": {"max_tokens": 5},
        "stream": True,
    }
    async with client.stream("POST", GEN_ENDPOINT, json=payload) as resp:
        resp.raise_for_status()
        chunks = []
        async for line in resp.aiter_lines():
            if not line.startswith("data: "):
                continue
            payload_str = line[len("data: ") :]
            if payload_str == "[DONE]":
                break
            chunks.append(json.loads(payload_str))
# ... 20 lines omitted for brevity ...
            "token_ids": [1, 2, 3],
            "sampling_params": {"max_tokens": 5, "temperature": 0.0},
            "stream": False,
        },
    )
    non_stream_data = non_stream_resp.json()
    # Just verify we got the right number of tokens
    assert len(all_token_ids) == len(non_stream_data["choices"][0]["token_ids"])
```
**EN:** This async test validates `test_generate_stream`. Relevant pytest markers include `asyncio`. Key inputs are `client`. The main assertion is `len(chunks) > 0` and `chunks[-1]['choices'][0]['finish_reason'] is not None`.
**CN:** 这个异步测试验证 `test_generate_stream`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 核心断言是 `len(chunks) > 0` and `chunks[-1]['choices'][0]['finish_reason'] is not None`。

### Test / 测试: test_generate_logprobs (L195-L218)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("logprobs_value", [0, 1, 5])
async def test_generate_logprobs(client, logprobs_value):
    payload = {
        "model": MODEL_NAME,
        "token_ids": [1, 2, 3],
        "sampling_params": {
            "max_tokens": 5,
            "temperature": 0.0,
            "logprobs": logprobs_value,
        },
        "stream": False,
    }
    resp = await client.post(GEN_ENDPOINT, json=payload)
    resp.raise_for_status()
    data = resp.json()
    choice = data["choices"][0]
    assert choice["logprobs"] is not None
    logprobs_content = choice["logprobs"]["content"]
    assert len(logprobs_content) == len(choice["token_ids"])
    for entry in logprobs_content:
        assert "logprob" in entry
        assert len(entry["top_logprobs"]) >= 1
        assert len(entry["top_logprobs"]) == max(logprobs_value, 1)
```
**EN:** This async test validates `test_generate_logprobs`. It uses parameterization over `logprobs_value`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `logprobs_value`. The main assertion is `choice['logprobs'] is not None` and `len(logprobs_content) == len(choice['token_ids'])`.
**CN:** 这个异步测试验证 `test_generate_logprobs`。 它通过参数化组合 `logprobs_value`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`logprobs_value`。 核心断言是 `choice['logprobs'] is not None` and `len(logprobs_content) == len(choice['token_ids'])`。

### Test / 测试: test_same_response_as_chat_completions (L221-L289)
```python
@pytest.mark.asyncio
async def test_same_response_as_chat_completions(client, tokenizer, messages):
    token_ids = tokenizer.apply_chat_template(
        messages,
        add_generation_prompt=True,
        enable_thinking=False,  # default with Qwen3
        return_dict=True,  # default with Transformers v5
    ).input_ids

    for ignore_eos in [True, False]:
        payload = {
            "model": MODEL_NAME,
            "token_ids": token_ids,
            "sampling_params": {
                "max_tokens": 24,
                "temperature": 0.0,
                # NOTE coordinator will set this to skip detokenization
                "detokenize": False,
# ... 43 lines omitted for brevity ...
                gen_token_ids_truncated = gen_token_ids[:eos_pos]
                generate_res = tokenizer.decode(
                    gen_token_ids_truncated, skip_special_tokens=True
                )
                # Truncate completions_res to same length for comparison
                completions_res = completions_res[: len(generate_res)]

        assert generate_res == completions_res
```
**EN:** This async test validates `test_same_response_as_chat_completions`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `tokenizer`, `messages`. The main assertion is `generate_res == completions_res`.
**CN:** 这个异步测试验证 `test_same_response_as_chat_completions`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`tokenizer`、`messages`。 核心断言是 `generate_res == completions_res`。

### Test / 测试: test_stop_string_workflow (L292-L352)
```python
@pytest.mark.asyncio
async def test_stop_string_workflow(client, tokenizer, messages):
    token_ids = tokenizer.apply_chat_template(
        messages,
        add_generation_prompt=True,
        enable_thinking=False,  # default with Qwen3
        return_dict=True,  # default with Transformers v5
    ).input_ids
    payload = {
        "model": MODEL_NAME,
        "token_ids": token_ids,
        "sampling_params": {
            "max_tokens": 24,
            "temperature": 0.0,
            "detokenize": False,
            # stop strings are only supported when detokenize is True.
            "stop": ["27 member"],
        },
# ... 35 lines omitted for brevity ...
        "stream": False,
        "stop": ["27 member"],
        "chat_template_kwargs": dict(enable_thinking=False),
    }
    completions_resp = await client.post("/v1/chat/completions", json=payload)
    completions_data = completions_resp.json()
    completions_res = completions_data["choices"][0]["message"]["content"]
    assert generate_res == completions_res
```
**EN:** This async test validates `test_stop_string_workflow`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `tokenizer`, `messages`. It checks an expected failure path with `pytest.raises`. The main assertion is `stop_str == '27 member'` and `generate_res == completions_res`.
**CN:** 这个异步测试验证 `test_stop_string_workflow`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`tokenizer`、`messages`。 它使用 `pytest.raises` 检查预期失败路径。 核心断言是 `stop_str == '27 member'` and `generate_res == completions_res`。

### Test / 测试: test_generate_with_lora_adapter (L355-L425)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "server",
    [
        [
            "--enable-lora",
            "--lora-modules",
            "Alice=charent/self_cognition_Alice",
            "Bob=charent/self_cognition_Bob",
            "--max-lora-rank",
            "64",
            "--max-cpu-loras",
            "2",
        ]
    ],
    indirect=True,
)
async def test_generate_with_lora_adapter(client, tokenizer, messages):
# ... 45 lines omitted for brevity ...
        "stream": False,
        "chat_template_kwargs": dict(enable_thinking=False),
    }
    completions_resp = await client.post("/v1/chat/completions", json=payload)
    completions_data = completions_resp.json()
    completions_res = completions_data["choices"][0]["message"]["content"]

    assert generate_res == completions_res
```
**EN:** This async test validates `test_generate_with_lora_adapter`. It uses parameterization over `server`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `tokenizer`, `messages`. The main assertion is `{'Alice', 'Bob'}.issubset(models)` and `'choices' in data`.
**CN:** 这个异步测试验证 `test_generate_with_lora_adapter`。 它通过参数化组合 `server`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`tokenizer`、`messages`。 核心断言是 `{'Alice', 'Bob'}.issubset(models)` and `'choices' in data`。

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
- **Stdlib / 标准库**: `json`, `os`
- **Third-party / 第三方**: `httpx`, `pytest`, `pytest_asyncio`, `transformers.AutoTokenizer`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.config.ModelConfig`, `vllm.config.utils.getattr_iter`, `vllm.v1.engine.detokenizer.check_stop_strings`
