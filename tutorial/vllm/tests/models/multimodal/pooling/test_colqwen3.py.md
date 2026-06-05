# test_colqwen3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/pooling/test_colqwen3.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and multimodal processing. The file defines 6 test(s), 0 fixture(s), and 9 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与多模态处理。它定义了 6 个测试、0 个 fixture，以及 9 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L10-L23)
```python
from io import BytesIO

import pybase64 as base64
import pytest
import torch
from PIL import Image

from vllm.entrypoints.chat_utils import (
    ChatCompletionContentPartImageParam,
    ChatCompletionContentPartTextParam,
)
from vllm.entrypoints.pooling.scoring.typing import ScoreMultiModalParam

from ....conftest import VllmRunner
```
**EN:** Imports standard-library modules such as `io.BytesIO`, third-party packages like `PIL.Image`, `pybase64`, `pytest`, project helpers such as `vllm.entrypoints.chat_utils.ChatCompletionContentPartImageParam`, `vllm.entrypoints.chat_utils.ChatCompletionContentPartTextParam`, `vllm.entrypoints.pooling.scoring.typing.ScoreMultiModalParam`.
**CN:** 导入标准库模块（如 `io.BytesIO`）、第三方包（如 `PIL.Image`、`pybase64`、`pytest`）、项目内辅助模块（如 `vllm.entrypoints.chat_utils.ChatCompletionContentPartImageParam`、`vllm.entrypoints.chat_utils.ChatCompletionContentPartTextParam`、`vllm.entrypoints.pooling.scoring.typing.ScoreMultiModalParam`）。

### Module setup / 模块级配置: pytestmark, MODELS, EMBED_DIMS (L25-L53)
```python
pytestmark = pytest.mark.skip(
    reason="ColQwen3 model's weight tying is incompatible with "
    "transformers v5 (missing all_tied_weights_keys)"
)

MODELS = [
    "TomoroAI/tomoro-colqwen3-embed-4b",
    "OpenSearch-AI/Ops-Colqwen3-4B",
    "nvidia/nemotron-colembed-vl-4b-v2",
]

EMBED_DIMS = {
    "TomoroAI/tomoro-colqwen3-embed-4b": 320,
    "OpenSearch-AI/Ops-Colqwen3-4B": 2560,
    "nvidia/nemotron-colembed-vl-4b-v2": 2560,
}
# ... 7 lines omitted for brevity ...
    "The capital of France is Paris.",
    "This document contains important financial data.",
]

DTYPE = "half"
GPU_MEMORY_UTILIZATION = 0.7
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `pytestmark`, `MODELS`, `EMBED_DIMS`, `TEXT_QUERIES`, `TEXT_DOCUMENTS`, `DTYPE`, `GPU_MEMORY_UTILIZATION`. Later helpers and tests reuse these values to keep scenarios concise. It also applies module-wide pytest markers.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `pytestmark`、`MODELS`、`EMBED_DIMS`、`TEXT_QUERIES`、`TEXT_DOCUMENTS`、`DTYPE`、`GPU_MEMORY_UTILIZATION`。后续辅助函数和测试会复用这些值，以减少重复。 它还会应用模块级的 pytest 标记。

### Helper / 辅助函数: _make_base64_image (L56-L64)
```python
def _make_base64_image(
    width: int = 64, height: int = 64, color: tuple[int, int, int] = (255, 0, 0)
) -> str:
    """Create a small solid-color PNG image and return its base64 data URI."""
    img = Image.new("RGB", (width, height), color)
    buf = BytesIO()
    img.save(buf, format="PNG")
    b64 = base64.b64encode(buf.getvalue()).decode()
    return f"data:image/png;base64,{b64}"
```
**EN:** This helper encapsulates reusable logic in `_make_base64_image`. Key inputs are `width`, `height`, `color`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_make_base64_image` 中。 关键输入包括 `width`、`height`、`color`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _make_image_mm_param (L67-L82)
```python
def _make_image_mm_param(
    image_uri: str,
    text: str | None = None,
) -> ScoreMultiModalParam:
    """Build a ScoreMultiModalParam containing an image (and optional text)."""
    content: list = [
        ChatCompletionContentPartImageParam(
            type="image_url",
            image_url={"url": image_uri},
        ),
    ]
    if text is not None:
        content.append(
            ChatCompletionContentPartTextParam(type="text", text=text),
        )
    return ScoreMultiModalParam(content=content)
```
**EN:** This helper encapsulates reusable logic in `_make_image_mm_param`. Key inputs are `image_uri`, `text`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_make_image_mm_param` 中。 关键输入包括 `image_uri`、`text`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _make_text_mm_param (L85-L89)
```python
def _make_text_mm_param(text: str) -> ScoreMultiModalParam:
    """Build a ScoreMultiModalParam containing only text."""
    return ScoreMultiModalParam(
        content=[ChatCompletionContentPartTextParam(type="text", text=text)],
    )
```
**EN:** This helper encapsulates reusable logic in `_make_text_mm_param`. Key inputs are `text`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_make_text_mm_param` 中。 关键输入包括 `text`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _run_token_embed_test (L92-L123)
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
        gpu_memory_utilization=GPU_MEMORY_UTILIZATION,
    ) as vllm_model:
        outputs = vllm_model.token_embed([TEXT_QUERIES[0]])

        assert len(outputs) == 1
# ... 6 lines omitted for brevity ...
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

### Helper / 辅助函数: _run_late_interaction_test (L126-L154)
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
        gpu_memory_utilization=GPU_MEMORY_UTILIZATION,
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

### Helper / 辅助函数: _run_relevance_test (L157-L183)
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
        gpu_memory_utilization=GPU_MEMORY_UTILIZATION,
    ) as vllm_model:
        scores = vllm_model.score(query, documents)

        assert len(scores) == 3
        assert scores[0] > scores[1], "ML doc should score higher than weather doc"
        assert scores[2] > scores[1], "DL doc should score higher than weather doc"
```
**EN:** This helper encapsulates reusable logic in `_run_relevance_test`. Key inputs are `vllm_runner`, `model`. The main assertion is `len(scores) == 3` and `scores[0] > scores[1]`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_relevance_test` 中。 关键输入包括 `vllm_runner`、`model`。 核心断言是 `len(scores) == 3` and `scores[0] > scores[1]`。

### Test / 测试: test_colqwen3_token_embed (L186-L193)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", [DTYPE])
def test_colqwen3_token_embed(
    vllm_runner,
    model: str,
    dtype: str,
) -> None:
    _run_token_embed_test(vllm_runner, model, dtype=dtype)
```
**EN:** This test validates `test_colqwen3_token_embed`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `model`, `dtype`.
**CN:** 这个测试验证 `test_colqwen3_token_embed`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`model`、`dtype`。

### Test / 测试: test_colqwen3_late_interaction_scoring (L196-L203)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", [DTYPE])
def test_colqwen3_late_interaction_scoring(
    vllm_runner,
    model: str,
    dtype: str,
) -> None:
    _run_late_interaction_test(vllm_runner, model, dtype=dtype)
```
**EN:** This test validates `test_colqwen3_late_interaction_scoring`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `model`, `dtype`.
**CN:** 这个测试验证 `test_colqwen3_late_interaction_scoring`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`model`、`dtype`。

### Test / 测试: test_colqwen3_relevance_ordering (L206-L213)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", [DTYPE])
def test_colqwen3_relevance_ordering(
    vllm_runner,
    model: str,
    dtype: str,
) -> None:
    _run_relevance_test(vllm_runner, model, dtype=dtype)
```
**EN:** This test validates `test_colqwen3_relevance_ordering`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `model`, `dtype`.
**CN:** 这个测试验证 `test_colqwen3_relevance_ordering`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`model`、`dtype`。

### Helper / 辅助函数: _run_multimodal_text_query_image_docs_test (L219-L251)
```python
def _run_multimodal_text_query_image_docs_test(
    vllm_runner: type[VllmRunner],
    model: str,
    *,
    dtype: str,
) -> None:
    """Score a text query against image documents via the multimodal path.

    Verifies that score_data_to_prompts correctly handles image content
    and produces valid MaxSim scores.
    """
    red_image = _make_base64_image(64, 64, color=(255, 0, 0))
    blue_image = _make_base64_image(64, 64, color=(0, 0, 255))

    query = "Describe the red object"
    image_docs = [
        _make_image_mm_param(red_image),
        _make_image_mm_param(blue_image),
# ... 7 lines omitted for brevity ...
        enforce_eager=True,
        gpu_memory_utilization=GPU_MEMORY_UTILIZATION,
    ) as vllm_model:
        scores = vllm_model.llm.score(query, image_docs)

        assert len(scores) == 2
        for s in scores:
            assert isinstance(s.outputs.score, float)
```
**EN:** This helper encapsulates reusable logic in `_run_multimodal_text_query_image_docs_test`. Key inputs are `vllm_runner`, `model`. The main assertion is `len(scores) == 2` and `isinstance(s.outputs.score, float)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_multimodal_text_query_image_docs_test` 中。 关键输入包括 `vllm_runner`、`model`。 核心断言是 `len(scores) == 2` and `isinstance(s.outputs.score, float)`。

### Helper / 辅助函数: _run_multimodal_mixed_docs_test (L254-L288)
```python
def _run_multimodal_mixed_docs_test(
    vllm_runner: type[VllmRunner],
    model: str,
    *,
    dtype: str,
) -> None:
    """Score a text query against a mix of text and image documents.

    Ensures the late-interaction path handles heterogeneous document
    types (plain strings alongside ScoreMultiModalParam images) in
    a single call.
    """
    red_image = _make_base64_image(64, 64, color=(255, 0, 0))

    query = "What is the capital of France?"
    documents: list = [
        "The capital of France is Paris.",
        _make_image_mm_param(red_image),
# ... 9 lines omitted for brevity ...
    ) as vllm_model:
        scores = vllm_model.llm.score(query, documents)

        assert len(scores) == 2
        for s in scores:
            assert isinstance(s.outputs.score, float)
        # Text document about France should score higher than a random image
        assert scores[0].outputs.score > scores[1].outputs.score
```
**EN:** This helper encapsulates reusable logic in `_run_multimodal_mixed_docs_test`. Key inputs are `vllm_runner`, `model`. The main assertion is `len(scores) == 2` and `scores[0].outputs.score > scores[1].outputs.score`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_multimodal_mixed_docs_test` 中。 关键输入包括 `vllm_runner`、`model`。 核心断言是 `len(scores) == 2` and `scores[0].outputs.score > scores[1].outputs.score`。

### Helper / 辅助函数: _run_multimodal_image_query_text_docs_test (L291-L322)
```python
def _run_multimodal_image_query_text_docs_test(
    vllm_runner: type[VllmRunner],
    model: str,
    *,
    dtype: str,
) -> None:
    """Score an image query against text documents.

    Verifies the reverse direction: multimodal query with text-only
    documents through the late-interaction scoring path.
    """
    red_image = _make_base64_image(64, 64, color=(255, 0, 0))
    image_query = _make_image_mm_param(red_image, text="red color")

    documents = [
        "A bright red sports car.",
        "The weather forecast shows rain tomorrow.",
    ]
# ... 6 lines omitted for brevity ...
        enforce_eager=True,
        gpu_memory_utilization=GPU_MEMORY_UTILIZATION,
    ) as vllm_model:
        scores = vllm_model.llm.score(image_query, documents)

        assert len(scores) == 2
        for s in scores:
            assert isinstance(s.outputs.score, float)
```
**EN:** This helper encapsulates reusable logic in `_run_multimodal_image_query_text_docs_test`. Key inputs are `vllm_runner`, `model`. The main assertion is `len(scores) == 2` and `isinstance(s.outputs.score, float)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_multimodal_image_query_text_docs_test` 中。 关键输入包括 `vllm_runner`、`model`。 核心断言是 `len(scores) == 2` and `isinstance(s.outputs.score, float)`。

### Test / 测试: test_colqwen3_multimodal_text_query_image_docs (L325-L332)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", [DTYPE])
def test_colqwen3_multimodal_text_query_image_docs(
    vllm_runner,
    model: str,
    dtype: str,
) -> None:
    _run_multimodal_text_query_image_docs_test(vllm_runner, model, dtype=dtype)
```
**EN:** This test validates `test_colqwen3_multimodal_text_query_image_docs`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `model`, `dtype`.
**CN:** 这个测试验证 `test_colqwen3_multimodal_text_query_image_docs`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`model`、`dtype`。

### Test / 测试: test_colqwen3_multimodal_mixed_docs (L335-L342)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", [DTYPE])
def test_colqwen3_multimodal_mixed_docs(
    vllm_runner,
    model: str,
    dtype: str,
) -> None:
    _run_multimodal_mixed_docs_test(vllm_runner, model, dtype=dtype)
```
**EN:** This test validates `test_colqwen3_multimodal_mixed_docs`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `model`, `dtype`.
**CN:** 这个测试验证 `test_colqwen3_multimodal_mixed_docs`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`model`、`dtype`。

### Test / 测试: test_colqwen3_multimodal_image_query_text_docs (L345-L352)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", [DTYPE])
def test_colqwen3_multimodal_image_query_text_docs(
    vllm_runner,
    model: str,
    dtype: str,
) -> None:
    _run_multimodal_image_query_text_docs_test(vllm_runner, model, dtype=dtype)
```
**EN:** This test validates `test_colqwen3_multimodal_image_query_text_docs`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `model`, `dtype`.
**CN:** 这个测试验证 `test_colqwen3_multimodal_image_query_text_docs`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`model`、`dtype`。

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
- **Stdlib / 标准库**: `io.BytesIO`
- **Third-party / 第三方**: `PIL.Image`, `pybase64`, `pytest`, `torch`
- **Project / 项目内**: `vllm.entrypoints.chat_utils.ChatCompletionContentPartImageParam`, `vllm.entrypoints.chat_utils.ChatCompletionContentPartTextParam`, `vllm.entrypoints.pooling.scoring.typing.ScoreMultiModalParam`, `vllm.entrypoints.pooling.scoring.utils.compute_maxsim_score`
- **Local relative imports / 本地相对导入**: `....conftest.VllmRunner`
