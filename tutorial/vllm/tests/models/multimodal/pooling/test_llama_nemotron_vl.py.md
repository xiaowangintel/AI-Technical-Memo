# test_llama_nemotron_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/pooling/test_llama_nemotron_vl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and multimodal processing. The file defines 4 test(s), 0 fixture(s), and 5 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与多模态处理。它定义了 4 个测试、0 个 fixture，以及 5 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L12-L29)
```python
from io import BytesIO
from pathlib import Path

import pybase64 as base64
import pytest
import torch
from transformers import AutoModel, AutoModelForSequenceClassification, AutoProcessor

from vllm.entrypoints.chat_utils import (
    ChatCompletionContentPartImageParam,
    ChatCompletionContentPartTextParam,
)
from vllm.entrypoints.pooling.scoring.typing import ScoreMultiModalParam
from vllm.platforms import current_platform

from ....conftest import IMAGE_ASSETS, HfRunner, PromptImageInput, VllmRunner
from ....utils import ROCM_ENGINE_KWARGS
from ...utils import check_embeddings_close
```
**EN:** Imports standard-library modules such as `io.BytesIO`, `pathlib.Path`, third-party packages like `pybase64`, `pytest`, `torch`, project helpers such as `vllm.entrypoints.chat_utils.ChatCompletionContentPartImageParam`, `vllm.entrypoints.chat_utils.ChatCompletionContentPartTextParam`, `vllm.entrypoints.pooling.scoring.typing.ScoreMultiModalParam`.
**CN:** 导入标准库模块（如 `io.BytesIO`、`pathlib.Path`）、第三方包（如 `pybase64`、`pytest`、`torch`）、项目内辅助模块（如 `vllm.entrypoints.chat_utils.ChatCompletionContentPartImageParam`、`vllm.entrypoints.chat_utils.ChatCompletionContentPartTextParam`、`vllm.entrypoints.pooling.scoring.typing.ScoreMultiModalParam`）。

### Module setup / 模块级配置: QUERY_PREFIX, PASSAGE_PREFIX, HF_TEXT_PROMPTS (L32-L51)
```python
QUERY_PREFIX = "query: "
PASSAGE_PREFIX = "passage: "

# Text prompts for text-only embedding
HF_TEXT_PROMPTS = [
    # T -> X (text embedding queries)
    f"{QUERY_PREFIX}The label of the object is stop sign",
    f"{QUERY_PREFIX}cherry blossom",
]

# Image prompts using the model's expected format
HF_IMAGE_PROMPTS = IMAGE_ASSETS.prompts(
    {
        # I -> X (image embedding as passage/document)
        "stop_sign": f"{PASSAGE_PREFIX}<image>",
        "cherry_blossom": f"{PASSAGE_PREFIX}<image>",
    }
)

MODELS = ["nvidia/llama-nemotron-embed-vl-1b-v2"]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `QUERY_PREFIX`, `PASSAGE_PREFIX`, `HF_TEXT_PROMPTS`, `HF_IMAGE_PROMPTS`, `MODELS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `QUERY_PREFIX`、`PASSAGE_PREFIX`、`HF_TEXT_PROMPTS`、`HF_IMAGE_PROMPTS`、`MODELS`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _run_test (L54-L114)
```python
def _run_test(
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    input_texts: list[str],
    input_images: PromptImageInput,
    model: str,
    *,
    dtype: str,
) -> None:
    """Run embedding comparison test between HF and vLLM.

    NOTE: Run vLLM first to avoid CUDA initialization issues with multiprocessing.
    """
    # Run vLLM inference first
    with vllm_runner(
        model,
        runner="pooling",
        dtype=dtype,
# ... 35 lines omitted for brevity ...
                hf_outputs.append(embedding[0].tolist())

    check_embeddings_close(
        embeddings_0_lst=hf_outputs,
        embeddings_1_lst=vllm_outputs,
        name_0="hf",
        name_1="vllm",
    )
```
**EN:** This helper encapsulates reusable logic in `_run_test`. Key inputs are `hf_runner`, `vllm_runner`, `input_texts`, `input_images`, `model`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_test` 中。 关键输入包括 `hf_runner`、`vllm_runner`、`input_texts`、`input_images`、`model`。

### Test / 测试: test_models_text (L117-L138)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["half"])
def test_models_text(
    hf_runner,
    vllm_runner,
    image_assets,
    model: str,
    dtype: str,
) -> None:
    """Test text-only embedding."""
    input_texts_images = [(text, None) for text in HF_TEXT_PROMPTS]
    input_texts = [text for text, _ in input_texts_images]
    input_images = [image for _, image in input_texts_images]

    _run_test(
        hf_runner,
        vllm_runner,
        input_texts,
        input_images,  # type: ignore
        model,
        dtype=dtype,
    )
```
**EN:** This test validates `test_models_text`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `image_assets`, `model`, `dtype`.
**CN:** 这个测试验证 `test_models_text`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`image_assets`、`model`、`dtype`。

### Test / 测试: test_models_image (L141-L164)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["half"])
def test_models_image(
    hf_runner,
    vllm_runner,
    image_assets,
    model: str,
    dtype: str,
) -> None:
    """Test image embedding."""
    input_texts_images = [
        (text, asset.pil_image) for text, asset in zip(HF_IMAGE_PROMPTS, image_assets)
    ]
    input_texts = [text for text, _ in input_texts_images]
    input_images = [image for _, image in input_texts_images]

    _run_test(
        hf_runner,
        vllm_runner,
        input_texts,
        input_images,
        model,
        dtype=dtype,
    )
```
**EN:** This test validates `test_models_image`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `image_assets`, `model`, `dtype`.
**CN:** 这个测试验证 `test_models_image`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`image_assets`、`model`、`dtype`。

### Module setup / 模块级配置: RERANKER_MODELS, _RERANKER_SCORE_TEMPLATE, RERANKER_TEXT_QUERY (L171-L189)
```python
RERANKER_MODELS = ["nvidia/llama-nemotron-rerank-vl-1b-v2"]

# The tokenizer's built-in chat template is not suitable for the Score/Rerank
# APIs (it's inherited from the base LLM).  We must use the provided override.
_RERANKER_SCORE_TEMPLATE = (
    Path(__file__).parents[4]
    / "examples/pooling/score/template/nemotron-vl-rerank.jinja"
).read_text()

RERANKER_TEXT_QUERY = "How is AI improving the intelligence and capabilities of robots?"
RERANKER_TEXT_DOCS = [
    "AI enables robots to perceive, plan, and act autonomously.",
    (
        "A biological foundation model designed to analyze DNA, RNA, "
        "and protein sequences."
    ),
]

RERANKER_IMAGE_QUERY = "photo of a red stop sign on a street"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `RERANKER_MODELS`, `_RERANKER_SCORE_TEMPLATE`, `RERANKER_TEXT_QUERY`, `RERANKER_TEXT_DOCS`, `RERANKER_IMAGE_QUERY`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `RERANKER_MODELS`、`_RERANKER_SCORE_TEMPLATE`、`RERANKER_TEXT_QUERY`、`RERANKER_TEXT_DOCS`、`RERANKER_IMAGE_QUERY`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _pil_to_data_uri (L192-L196)
```python
def _pil_to_data_uri(image) -> str:
    buf = BytesIO()
    image.save(buf, format="PNG")
    b64 = base64.b64encode(buf.getvalue()).decode()
    return f"data:image/png;base64,{b64}"
```
**EN:** This helper encapsulates reusable logic in `_pil_to_data_uri`. Key inputs are `image`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_pil_to_data_uri` 中。 关键输入包括 `image`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _run_hf_reranker (L199-L238)
```python
def _run_hf_reranker(
    hf_runner: type[HfRunner],
    model: str,
    dtype: str,
    query: str,
    docs: list,
) -> list[float]:
    """Run HF reranker inference; docs is a list of (doc_text, doc_image|None)."""
    with hf_runner(
        model,
        dtype=dtype,
        trust_remote_code=True,
        auto_cls=AutoModelForSequenceClassification,
    ) as hf_model:
        processor = AutoProcessor.from_pretrained(
            model,
            trust_remote_code=True,
            max_input_tiles=6,
# ... 14 lines omitted for brevity ...
            for k, v in batch_dict.items()
        }
        with torch.inference_mode():
            logits = hf_model.model(**batch_dict, return_dict=True).logits
        # vLLM applies sigmoid activation to the raw logits before returning
        # scores; apply the same here so both sides are comparable.
        scores = torch.sigmoid(logits.squeeze(-1).float())
        return scores.detach().cpu().tolist()
```
**EN:** This helper encapsulates reusable logic in `_run_hf_reranker`. Key inputs are `hf_runner`, `model`, `dtype`, `query`, `docs`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_hf_reranker` 中。 关键输入包括 `hf_runner`、`model`、`dtype`、`query`、`docs`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _run_vllm_reranker (L241-L308)
```python
def _run_vllm_reranker(
    vllm_runner: type[VllmRunner],
    model: str,
    dtype: str,
    query: str,
    docs: list,
) -> list[float]:
    """Run vLLM reranker inference; docs is a list of (doc_text, doc_image|None)."""
    with vllm_runner(
        model,
        runner="pooling",
        dtype=dtype,
        max_model_len=2048,
        enforce_eager=True,
        trust_remote_code=True,
        **ROCM_ENGINE_KWARGS,
    ) as vllm_model:
        has_images = any(img is not None for _, img in docs)
# ... 42 lines omitted for brevity ...
            raw_outputs = vllm_model.llm.score(
                query_params,
                doc_params,
                chat_template=_RERANKER_SCORE_TEMPLATE,
            )
            outputs = [o.outputs.score for o in raw_outputs]

    return outputs
```
**EN:** This helper encapsulates reusable logic in `_run_vllm_reranker`. Key inputs are `vllm_runner`, `model`, `dtype`, `query`, `docs`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_vllm_reranker` 中。 关键输入包括 `vllm_runner`、`model`、`dtype`、`query`、`docs`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _run_reranker_test (L311-L335)
```python
def _run_reranker_test(
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    model: str,
    dtype: str,
    query: str,
    docs: list,
) -> None:
    """Compare HF and vLLM reranker scores.

    NOTE: Run vLLM first to avoid CUDA initialization issues with multiprocessing.
    """
    vllm_scores = _run_vllm_reranker(vllm_runner, model, dtype, query, docs)
    hf_scores = _run_hf_reranker(hf_runner, model, dtype, query, docs)

    assert len(hf_scores) == len(vllm_scores), (
        f"Output length mismatch: HF={len(hf_scores)}, vLLM={len(vllm_scores)}"
    )
    # NOTE: ROCm shows slightly higher numerical variance dues to different attention
    # backend between vLLM and HF; use a marginally looser tolerance
    rel_tol = 0.022 if current_platform.is_rocm() else 0.02
    for i, (hf_score, vllm_score) in enumerate(zip(hf_scores, vllm_scores)):
        assert hf_score == pytest.approx(vllm_score, rel=rel_tol), (
            f"Score mismatch at index {i}: HF={hf_score:.4f}, vLLM={vllm_score:.4f}"
        )
```
**EN:** This helper encapsulates reusable logic in `_run_reranker_test`. Key inputs are `hf_runner`, `vllm_runner`, `model`, `dtype`, `query`, `docs`. The main assertion is `len(hf_scores) == len(vllm_scores)` and `hf_score == pytest.approx(vllm_score, rel=rel_tol)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_reranker_test` 中。 关键输入包括 `hf_runner`、`vllm_runner`、`model`、`dtype`、`query`、`docs`。 核心断言是 `len(hf_scores) == len(vllm_scores)` and `hf_score == pytest.approx(vllm_score, rel=rel_tol)`。

### Test / 测试: test_reranker_text (L338-L348)
```python
@pytest.mark.parametrize("model", RERANKER_MODELS)
@pytest.mark.parametrize("dtype", ["half"])
def test_reranker_text(
    hf_runner,
    vllm_runner,
    model: str,
    dtype: str,
) -> None:
    """Test reranking with text-only query and text documents."""
    docs = [(text, None) for text in RERANKER_TEXT_DOCS]
    _run_reranker_test(hf_runner, vllm_runner, model, dtype, RERANKER_TEXT_QUERY, docs)
```
**EN:** This test validates `test_reranker_text`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `model`, `dtype`.
**CN:** 这个测试验证 `test_reranker_text`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`model`、`dtype`。

### Test / 测试: test_reranker_image_doc (L351-L362)
```python
@pytest.mark.parametrize("model", RERANKER_MODELS)
@pytest.mark.parametrize("dtype", ["half"])
def test_reranker_image_doc(
    hf_runner,
    vllm_runner,
    image_assets,
    model: str,
    dtype: str,
) -> None:
    """Test reranking with text query against image documents."""
    docs = [(None, asset.pil_image) for asset in image_assets]
    _run_reranker_test(hf_runner, vllm_runner, model, dtype, RERANKER_IMAGE_QUERY, docs)
```
**EN:** This test validates `test_reranker_image_doc`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `image_assets`, `model`, `dtype`.
**CN:** 这个测试验证 `test_reranker_image_doc`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`image_assets`、`model`、`dtype`。

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
- **Stdlib / 标准库**: `io.BytesIO`, `pathlib.Path`
- **Third-party / 第三方**: `pybase64`, `pytest`, `torch`, `transformers.AutoModel`, `transformers.AutoModelForSequenceClassification`, `transformers.AutoProcessor`
- **Project / 项目内**: `vllm.entrypoints.chat_utils.ChatCompletionContentPartImageParam`, `vllm.entrypoints.chat_utils.ChatCompletionContentPartTextParam`, `vllm.entrypoints.pooling.scoring.typing.ScoreMultiModalParam`, `vllm.platforms.current_platform`
- **Local relative imports / 本地相对导入**: `....conftest.HfRunner`, `....conftest.IMAGE_ASSETS`, `....conftest.PromptImageInput`, `....conftest.VllmRunner`, `....utils.ROCM_ENGINE_KWARGS`, `...utils.check_embeddings_close`
