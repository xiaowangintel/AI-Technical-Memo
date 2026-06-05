# embed_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/pooling/embed_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared test utilities for pooling or embedding behavior and language-model behavior. The file exposes 3 helper/class block(s) used by nearby tests. / [CN] 为池化或嵌入行为与语言模型行为提供共享测试工具。该文件暴露了 3 个辅助函数/类块，供附近测试复用。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L9)
```python
from collections.abc import Sequence

import openai
import pytest

from tests.conftest import HfRunner
from tests.models.utils import EmbedModelInfo, check_embeddings_close, matryoshka_fy
```
**EN:** Imports standard-library modules such as `collections.abc.Sequence`, third-party packages like `openai`, `pytest`, project helpers such as `tests.conftest.HfRunner`, `tests.models.utils.EmbedModelInfo`, `tests.models.utils.check_embeddings_close`.
**CN:** 导入标准库模块（如 `collections.abc.Sequence`）、第三方包（如 `openai`、`pytest`）、项目内辅助模块（如 `tests.conftest.HfRunner`、`tests.models.utils.EmbedModelInfo`、`tests.models.utils.check_embeddings_close`）。

### Helper / 辅助函数: run_embedding_correctness_test (L12-L28)
```python
def run_embedding_correctness_test(
    hf_model: "HfRunner",
    inputs: list[str],
    vllm_outputs: Sequence[list[float]],
    dimensions: int | None = None,
):
    hf_outputs = hf_model.encode(inputs)
    if dimensions:
        hf_outputs = matryoshka_fy(hf_outputs, dimensions)

    check_embeddings_close(
        embeddings_0_lst=hf_outputs,
        embeddings_1_lst=vllm_outputs,
        name_0="hf",
        name_1="vllm",
        tol=1e-2,
    )
```
**EN:** This helper encapsulates reusable logic in `run_embedding_correctness_test`. Key inputs are `hf_model`, `inputs`, `vllm_outputs`, `dimensions`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_embedding_correctness_test` 中。 关键输入包括 `hf_model`、`inputs`、`vllm_outputs`、`dimensions`。

### Helper / 辅助函数: correctness_test_embed_models (L31-L68)
```python
def correctness_test_embed_models(
    hf_runner,
    vllm_runner,
    model_info: EmbedModelInfo,
    example_prompts,
    vllm_extra_kwargs=None,
    hf_model_callback=None,
):
    pytest.skip("Debug only, ci prefers to use mteb test.")

    # The example_prompts has ending "\n", for example:
    # "Write a short story about a robot that dreams for the first time.\n"
    # sentence_transformers will strip the input texts, see:
    # https://github.com/UKPLab/sentence-transformers/blob/v3.1.1/sentence_transformers/models/Transformer.py#L159
    # This makes the input_ids different between hf_model and vllm_model.
    # So we need to strip the input texts to avoid test failing.
    example_prompts = [str(s).strip() for s in example_prompts]

# ... 12 lines omitted for brevity ...
        model_info.name,
        dtype=model_info.hf_dtype,
        is_sentence_transformer=True,
    ) as hf_model:
        if hf_model_callback is not None:
            hf_model_callback(hf_model)

        run_embedding_correctness_test(hf_model, example_prompts, vllm_outputs)
```
**EN:** This helper encapsulates reusable logic in `correctness_test_embed_models`. Key inputs are `hf_runner`, `vllm_runner`, `model_info`, `example_prompts`, `vllm_extra_kwargs`, `hf_model_callback`. The logic can skip unsupported environments when prerequisites are not satisfied.
**CN:** 这个辅助函数将可复用逻辑封装在 `correctness_test_embed_models` 中。 关键输入包括 `hf_runner`、`vllm_runner`、`model_info`、`example_prompts`、`vllm_extra_kwargs`、`hf_model_callback`。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。

### Helper / 辅助函数: run_client_embeddings (L71-L81)
```python
async def run_client_embeddings(
    client: openai.AsyncOpenAI,
    model_name: str,
    queries: list[str],
    instruction: str = "",
) -> list[list[float]]:
    outputs = await client.embeddings.create(
        model=model_name,
        input=[instruction + q for q in queries],
    )
    return [data.embedding for data in outputs.data]
```
**EN:** This async helper encapsulates reusable logic in `run_client_embeddings`. Key inputs are `client`, `model_name`, `queries`, `instruction`. It drives client-facing request creation through the API surface under test. It returns computed state or helper objects back to the caller.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `run_client_embeddings` 中。 关键输入包括 `client`、`model_name`、`queries`、`instruction`。 它通过被测 API 表面触发面向客户端的请求创建流程。 它把计算得到的状态或辅助对象返回给调用方。

## Key Concepts / 关键概念
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Shared metadata and helper utilities keep nearby tests focused on behavior.
  **CN:** 共享元数据与辅助工具让相邻测试更专注于行为本身。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc.Sequence`
- **Third-party / 第三方**: `openai`, `pytest`
- **Project / 项目内**: `tests.conftest.HfRunner`, `tests.models.utils.EmbedModelInfo`, `tests.models.utils.check_embeddings_close`, `tests.models.utils.matryoshka_fy`
