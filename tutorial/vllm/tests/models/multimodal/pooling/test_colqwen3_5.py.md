# test_colqwen3_5.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/pooling/test_colqwen3_5.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and multimodal processing. The file defines 3 test(s), 0 fixture(s), and 3 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与多模态处理。它定义了 3 个测试、0 个 fixture，以及 3 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L10-L13)
```python
import pytest
import torch

from ....conftest import VllmRunner
```
**EN:** Imports third-party packages like `pytest`, `torch`, project helpers such as `vllm.entrypoints.pooling.scoring.utils.compute_maxsim_score`, `....conftest.VllmRunner`.
**CN:** 导入第三方包（如 `pytest`、`torch`）、项目内辅助模块（如 `vllm.entrypoints.pooling.scoring.utils.compute_maxsim_score`、`....conftest.VllmRunner`）。

### Module setup / 模块级配置: MODELS, EMBED_DIMS, TEXT_QUERIES (L15-L33)
```python
MODELS = [
    "athrael-soju/colqwen3.5-4.5B-v3",
]

EMBED_DIMS = {
    "athrael-soju/colqwen3.5-4.5B-v3": 320,
}

TEXT_QUERIES = [
    "What is the capital of France?",
    "Describe the contents of the document.",
]

TEXT_DOCUMENTS = [
    "The capital of France is Paris.",
    "This document contains important financial data.",
]

DTYPE = "half"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODELS`, `EMBED_DIMS`, `TEXT_QUERIES`, `TEXT_DOCUMENTS`, `DTYPE`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODELS`、`EMBED_DIMS`、`TEXT_QUERIES`、`TEXT_DOCUMENTS`、`DTYPE`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _run_token_embed_test (L36-L66)
```python
def _run_token_embed_test(
    vllm_runner: type[VllmRunner],
    model: str,
    *,
    dtype: str,
) -> None:
    """Verify per-token embedding shape and L2 normalization."""
    with vllm_runner(
        model,
        runner="pooling",
        dtype=dtype,
        max_model_len=4096,
        enforce_eager=True,
    ) as vllm_model:
        outputs = vllm_model.token_embed([TEXT_QUERIES[0]])

        assert len(outputs) == 1
        emb = torch.tensor(outputs[0])
# ... 5 lines omitted for brevity ...
        # Verify L2 normalization
        norms = torch.norm(emb, p=2, dim=-1)
        torch.testing.assert_close(
            norms,
            torch.ones_like(norms),
            rtol=1e-2,
            atol=1e-2,
        )
```
**EN:** This helper encapsulates reusable logic in `_run_token_embed_test`. Key inputs are `vllm_runner`, `model`. The main assertion is `len(outputs) == 1` and `emb.dim() == 2`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_token_embed_test` 中。 关键输入包括 `vllm_runner`、`model`。 核心断言是 `len(outputs) == 1` and `emb.dim() == 2`。

### Helper / 辅助函数: _run_late_interaction_test (L69-L96)
```python
def _run_late_interaction_test(
    vllm_runner: type[VllmRunner],
    model: str,
    *,
    dtype: str,
) -> None:
    """Verify MaxSim scoring matches manual computation."""
    from vllm.entrypoints.pooling.scoring.utils import compute_maxsim_score

    with vllm_runner(
        model,
        runner="pooling",
        dtype=dtype,
        max_model_len=4096,
        enforce_eager=True,
    ) as vllm_model:
        q_outputs = vllm_model.token_embed([TEXT_QUERIES[0]])
        d_outputs = vllm_model.token_embed([TEXT_DOCUMENTS[0]])

        q_emb = torch.tensor(q_outputs[0])
        d_emb = torch.tensor(d_outputs[0])

        manual_score = compute_maxsim_score(q_emb, d_emb).item()

        vllm_scores = vllm_model.score(TEXT_QUERIES[0], TEXT_DOCUMENTS[0])

        assert len(vllm_scores) == 1
        assert vllm_scores[0] == pytest.approx(manual_score, rel=0.01)
```
**EN:** This helper encapsulates reusable logic in `_run_late_interaction_test`. Key inputs are `vllm_runner`, `model`. The main assertion is `len(vllm_scores) == 1` and `vllm_scores[0] == pytest.approx(manual_score, rel=0.01)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_late_interaction_test` 中。 关键输入包括 `vllm_runner`、`model`。 核心断言是 `len(vllm_scores) == 1` and `vllm_scores[0] == pytest.approx(manual_score, rel=0.01)`。

### Helper / 辅助函数: _run_relevance_test (L99-L124)
```python
def _run_relevance_test(
    vllm_runner: type[VllmRunner],
    model: str,
    *,
    dtype: str,
) -> None:
    """Verify that relevant documents score higher than irrelevant ones."""
    query = "What is machine learning?"
    documents = [
        "Machine learning is a subset of artificial intelligence.",
        "The weather forecast shows rain tomorrow.",
        "Deep learning uses neural networks for complex tasks.",
    ]

    with vllm_runner(
        model,
        runner="pooling",
        dtype=dtype,
        max_model_len=4096,
        enforce_eager=True,
    ) as vllm_model:
        scores = vllm_model.score(query, documents)

        assert len(scores) == 3
        assert scores[0] > scores[1], "ML doc should score higher than weather doc"
        assert scores[2] > scores[1], "DL doc should score higher than weather doc"
```
**EN:** This helper encapsulates reusable logic in `_run_relevance_test`. Key inputs are `vllm_runner`, `model`. The main assertion is `len(scores) == 3` and `scores[0] > scores[1]`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_relevance_test` 中。 关键输入包括 `vllm_runner`、`model`。 核心断言是 `len(scores) == 3` and `scores[0] > scores[1]`。

### Test / 测试: test_colqwen3_5_token_embed (L127-L134)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", [DTYPE])
def test_colqwen3_5_token_embed(
    vllm_runner,
    model: str,
    dtype: str,
) -> None:
    _run_token_embed_test(vllm_runner, model, dtype=dtype)
```
**EN:** This test validates `test_colqwen3_5_token_embed`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `model`, `dtype`.
**CN:** 这个测试验证 `test_colqwen3_5_token_embed`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`model`、`dtype`。

### Test / 测试: test_colqwen3_5_late_interaction_scoring (L137-L144)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", [DTYPE])
def test_colqwen3_5_late_interaction_scoring(
    vllm_runner,
    model: str,
    dtype: str,
) -> None:
    _run_late_interaction_test(vllm_runner, model, dtype=dtype)
```
**EN:** This test validates `test_colqwen3_5_late_interaction_scoring`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `model`, `dtype`.
**CN:** 这个测试验证 `test_colqwen3_5_late_interaction_scoring`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`model`、`dtype`。

### Test / 测试: test_colqwen3_5_relevance_ordering (L147-L154)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", [DTYPE])
def test_colqwen3_5_relevance_ordering(
    vllm_runner,
    model: str,
    dtype: str,
) -> None:
    _run_relevance_test(vllm_runner, model, dtype=dtype)
```
**EN:** This test validates `test_colqwen3_5_relevance_ordering`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `model`, `dtype`.
**CN:** 这个测试验证 `test_colqwen3_5_relevance_ordering`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`model`、`dtype`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `pytest`, `torch`
- **Project / 项目内**: `vllm.entrypoints.pooling.scoring.utils.compute_maxsim_score`
- **Local relative imports / 本地相对导入**: `....conftest.VllmRunner`
