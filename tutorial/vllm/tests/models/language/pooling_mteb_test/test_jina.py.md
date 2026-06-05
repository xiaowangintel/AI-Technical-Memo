# test_jina.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/pooling_mteb_test/test_jina.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and language-model behavior. The file defines 4 test(s), 0 fixture(s), and 2 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与语言模型行为。它定义了 4 个测试、0 个 fixture，以及 2 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L19)
```python
from functools import partial

import pytest

from tests.models.language.pooling.embed_utils import (
    check_embeddings_close,
    correctness_test_embed_models,
    matryoshka_fy,
)
from tests.models.utils import (
    EmbedModelInfo,
    RerankModelInfo,
)
from vllm import PoolingParams

from .mteb_embed_utils import mteb_test_embed_models
from .mteb_score_utils import mteb_test_rerank_models
```
**EN:** Imports standard-library modules such as `functools.partial`, third-party packages like `pytest`, project helpers such as `tests.models.language.pooling.embed_utils.check_embeddings_close`, `tests.models.language.pooling.embed_utils.correctness_test_embed_models`, `tests.models.language.pooling.embed_utils.matryoshka_fy`.
**CN:** 导入标准库模块（如 `functools.partial`）、第三方包（如 `pytest`）、项目内辅助模块（如 `tests.models.language.pooling.embed_utils.check_embeddings_close`、`tests.models.language.pooling.embed_utils.correctness_test_embed_models`、`tests.models.language.pooling.embed_utils.matryoshka_fy`）。

### Module setup / 模块级配置: EMBEDDING_MODELS, RERANK_MODELS (L21-L53)
```python
EMBEDDING_MODELS = [
    EmbedModelInfo(
        "jinaai/jina-embeddings-v3",
        mteb_score=0.824413164,
        architecture="XLMRobertaModel",
        is_matryoshka=True,
        seq_pooling_type="MEAN",
        attn_type="encoder_only",
        is_prefix_caching_supported=False,
        is_chunked_prefill_supported=False,
    ),
    EmbedModelInfo(
        "jinaai/jina-embeddings-v5-text-small",
        mteb_score=0.794535707854956,
        architecture="JinaEmbeddingsV5Model",
        seq_pooling_type="LAST",
# ... 11 lines omitted for brevity ...
        seq_pooling_type="CLS",
        attn_type="encoder_only",
        is_prefix_caching_supported=False,
        is_chunked_prefill_supported=False,
    )
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `EMBEDDING_MODELS`, `RERANK_MODELS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `EMBEDDING_MODELS`、`RERANK_MODELS`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_embed_models_mteb (L56-L70)
```python
@pytest.mark.parametrize("model_info", EMBEDDING_MODELS)
def test_embed_models_mteb(hf_runner, vllm_runner, model_info: EmbedModelInfo) -> None:
    task = "retrieval" if "v5" in model_info.name else "text-matching"
    prompt_prefix: str | None = "Document: " if "v5" in model_info.name else None

    def hf_model_callback(model):
        model.encode = partial(model.encode, task=task)

    mteb_test_embed_models(
        hf_runner,
        vllm_runner,
        model_info,
        hf_model_callback=hf_model_callback,
        prompt_prefix=prompt_prefix,
    )
```
**EN:** This test validates `test_embed_models_mteb`. It uses parameterization over `model_info`. Key inputs are `hf_runner`, `vllm_runner`, `model_info`.
**CN:** 这个测试验证 `test_embed_models_mteb`。 它通过参数化组合 `model_info`。 关键输入包括 `hf_runner`、`vllm_runner`、`model_info`。

### Test / 测试: test_embed_models_correctness (L73-L88)
```python
@pytest.mark.parametrize("model_info", EMBEDDING_MODELS)
def test_embed_models_correctness(
    hf_runner, vllm_runner, model_info: EmbedModelInfo, example_prompts
) -> None:
    task = "retrieval" if "v5" in model_info.name else "text-matching"

    def hf_model_callback(model):
        model.encode = partial(model.encode, task=task)

    correctness_test_embed_models(
        hf_runner,
        vllm_runner,
        model_info,
        example_prompts,
        hf_model_callback=hf_model_callback,
    )
```
**EN:** This test validates `test_embed_models_correctness`. It uses parameterization over `model_info`. Key inputs are `hf_runner`, `vllm_runner`, `model_info`, `example_prompts`.
**CN:** 这个测试验证 `test_embed_models_correctness`。 它通过参数化组合 `model_info`。 关键输入包括 `hf_runner`、`vllm_runner`、`model_info`、`example_prompts`。

### Test / 测试: test_rerank_models_mteb (L91-L93)
```python
@pytest.mark.parametrize("model_info", RERANK_MODELS)
def test_rerank_models_mteb(vllm_runner, model_info: RerankModelInfo) -> None:
    mteb_test_rerank_models(vllm_runner, model_info)
```
**EN:** This test validates `test_rerank_models_mteb`. It uses parameterization over `model_info`. Key inputs are `vllm_runner`, `model_info`.
**CN:** 这个测试验证 `test_rerank_models_mteb`。 它通过参数化组合 `model_info`。 关键输入包括 `vllm_runner`、`model_info`。

### Test / 测试: test_matryoshka (L96-L154)
```python
@pytest.mark.skip(
    reason="jinaai/jina-embeddings-v3 custom XLMRobertaLoRA model on HF hub "
    "is incompatible with transformers v5 (missing all_tied_weights_keys)"
)
@pytest.mark.parametrize("model_info", EMBEDDING_MODELS)
@pytest.mark.parametrize("dtype", ["half"])
@pytest.mark.parametrize("dimensions", [16, 32])
def test_matryoshka(
    hf_runner,
    vllm_runner,
    model_info,
    dtype: str,
    dimensions: int,
    example_prompts,
    monkeypatch,
) -> None:
    if not model_info.is_matryoshka:
        pytest.skip("Model is not matryoshka")
# ... 33 lines omitted for brevity ...

            check_embeddings_close(
                embeddings_0_lst=hf_outputs,
                embeddings_1_lst=vllm_outputs,
                name_0="hf",
                name_1="vllm",
                tol=1e-2,
            )
```
**EN:** This test validates `test_matryoshka`. It uses parameterization over `model_info`. Relevant pytest markers include `skip`. Key inputs are `hf_runner`, `vllm_runner`, `model_info`, `dtype`, `dimensions`, `example_prompts`. It checks an expected failure path with `pytest.raises`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The logic can skip unsupported environments when prerequisites are not satisfied. The main assertion is `vllm_model.llm.llm_engine.model_config.is_matryoshka` and `matryoshka_dimensions is not None`.
**CN:** 这个测试验证 `test_matryoshka`。 它通过参数化组合 `model_info`。 相关的 pytest 标记包括 `skip`。 关键输入包括 `hf_runner`、`vllm_runner`、`model_info`、`dtype`、`dimensions`、`example_prompts`。 它使用 `pytest.raises` 检查预期失败路径。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 核心断言是 `vllm_model.llm.llm_engine.model_config.is_matryoshka` and `matryoshka_dimensions is not None`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `functools.partial`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `tests.models.language.pooling.embed_utils.check_embeddings_close`, `tests.models.language.pooling.embed_utils.correctness_test_embed_models`, `tests.models.language.pooling.embed_utils.matryoshka_fy`, `tests.models.utils.EmbedModelInfo`, `tests.models.utils.RerankModelInfo`, `vllm.PoolingParams`
- **Local relative imports / 本地相对导入**: `.mteb_embed_utils.mteb_test_embed_models`, `.mteb_score_utils.mteb_test_rerank_models`
