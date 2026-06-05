# test_prompt_validation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/completion/test_prompt_validation.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers text completion behavior and OpenAI-compatible serving. The file defines 4 test(s), 0 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖文本补全行为与OpenAI 兼容服务。它定义了 4 个测试、0 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L16)
```python
import io
from unittest.mock import Mock

# imports for structured outputs tests
import openai
import pybase64
import pytest
import regex as re
import torch

from tests.utils import RemoteOpenAIServer
from vllm.config import ModelConfig
from vllm.renderers.embed_utils import safe_load_prompt_embeds
```
**EN:** Imports standard-library modules such as `io`, `unittest.mock.Mock`, third-party packages like `openai`, `pybase64`, `pytest`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.config.ModelConfig`, `vllm.renderers.embed_utils.safe_load_prompt_embeds`.
**CN:** 导入标准库模块（如 `io`、`unittest.mock.Mock`）、第三方包（如 `openai`、`pybase64`、`pytest`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.config.ModelConfig`、`vllm.renderers.embed_utils.safe_load_prompt_embeds`）。

### Test / 测试: test_empty_prompt (L19-L36)
```python
@pytest.mark.asyncio
async def test_empty_prompt():
    model_name = "gpt2"
    server_args = ["--enforce-eager"]
    with RemoteOpenAIServer(model_name, server_args) as remote_server:
        client = remote_server.get_async_client()

        with pytest.raises(
            openai.BadRequestError,
            match="Either prompt or prompt_embeds must be provided and non-empty.",
        ):
            await client.completions.create(
                model=model_name,
                prompt=None,
                max_tokens=5,
                temperature=0.0,
                extra_body={"prompt_embeds": []},
            )
```
**EN:** This async test validates `test_empty_prompt`. Relevant pytest markers include `asyncio`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_empty_prompt`。 相关的 pytest 标记包括 `asyncio`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_out_of_vocab_token_ids (L39-L51)
```python
@pytest.mark.asyncio
async def test_out_of_vocab_token_ids():
    model_name = "gpt2"
    server_args = ["--enforce-eager"]
    with RemoteOpenAIServer(model_name, server_args) as remote_server:
        client = remote_server.get_async_client()

        with pytest.raises(
            openai.BadRequestError, match=re.compile(".*out of vocabulary.*").pattern
        ):
            await client.completions.create(
                model=model_name, prompt=[999999], max_tokens=5, temperature=0.0
            )
```
**EN:** This async test validates `test_out_of_vocab_token_ids`. Relevant pytest markers include `asyncio`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_out_of_vocab_token_ids`。 相关的 pytest 标记包括 `asyncio`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_load_prompt_embeds (L54-L97)
```python
@pytest.mark.parametrize("dtype", [torch.float32, torch.bfloat16, torch.float16])
@pytest.mark.parametrize(
    "layout", [torch.strided, torch.sparse_coo, torch.sparse_csc, torch.sparse_csr]
)
@pytest.mark.parametrize("seq_len", [2, 10])
@pytest.mark.parametrize("hidden_size", [2, 10])
def test_load_prompt_embeds(
    dtype: torch.dtype, layout: torch.layout, seq_len: int, hidden_size: int
):
    model_config = Mock(spec=ModelConfig)
    model_config.enable_prompt_embeds = True
    model_config.get_hidden_size.return_value = hidden_size
    model_config.dtype = dtype

    # construct arbitrary tensors of various dtypes, layouts, and sizes.
    # We need to check against different layouts to make sure that if a user
    # uses sparse tensors to reduce the transmission size of prompt embeddings,
    # we must cast them to dense/strided before passing them into the engine.
# ... 18 lines omitted for brevity ...
    encoded_tensor = pybase64.b64encode(buffer.getvalue())

    loaded_tensor = safe_load_prompt_embeds(model_config, encoded_tensor)
    assert loaded_tensor.device.type == "cpu"
    assert loaded_tensor.layout == torch.strided
    torch.testing.assert_close(
        loaded_tensor, tensor.to("cpu").to_dense(), equal_nan=True
    )
```
**EN:** This test validates `test_load_prompt_embeds`. It uses parameterization over `dtype`. Key inputs are `dtype`, `layout`, `seq_len`, `hidden_size`. The main assertion is `loaded_tensor.device.type == 'cpu'` and `loaded_tensor.layout == torch.strided`.
**CN:** 这个测试验证 `test_load_prompt_embeds`。 它通过参数化组合 `dtype`。 关键输入包括 `dtype`、`layout`、`seq_len`、`hidden_size`。 核心断言是 `loaded_tensor.device.type == 'cpu'` and `loaded_tensor.layout == torch.strided`。

### Test / 测试: test_disable_prompt_embeds (L100-L115)
```python
@pytest.mark.parametrize("dtype", [torch.float32])
@pytest.mark.parametrize("seq_len", [2])
@pytest.mark.parametrize("hidden_size", [2])
def test_disable_prompt_embeds(dtype: torch.dtype, seq_len: int, hidden_size: int):
    model_config = Mock(spec=ModelConfig)
    model_config.enable_prompt_embeds = False

    tensor = torch.randn((seq_len, hidden_size), dtype=dtype)

    buffer = io.BytesIO()
    torch.save(tensor, buffer)
    buffer.seek(0)
    encoded_tensor = pybase64.b64encode(buffer.getvalue())

    with pytest.raises(ValueError, match="--enable-prompt-embeds"):
        safe_load_prompt_embeds(model_config, encoded_tensor)
```
**EN:** This test validates `test_disable_prompt_embeds`. It uses parameterization over `dtype`. Key inputs are `dtype`, `seq_len`, `hidden_size`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `test_disable_prompt_embeds`。 它通过参数化组合 `dtype`。 关键输入包括 `dtype`、`seq_len`、`hidden_size`。 它使用 `pytest.raises` 检查预期失败路径。

## Key Concepts / 关键概念
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `io`, `unittest.mock.Mock`
- **Third-party / 第三方**: `openai`, `pybase64`, `pytest`, `regex`, `torch`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.config.ModelConfig`, `vllm.renderers.embed_utils.safe_load_prompt_embeds`
