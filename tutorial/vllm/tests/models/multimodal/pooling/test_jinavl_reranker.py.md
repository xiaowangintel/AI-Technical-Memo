# test_jinavl_reranker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/pooling/test_jinavl_reranker.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and multimodal processing. The file defines 5 test(s), 0 fixture(s), and 5 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与多模态处理。它定义了 5 个测试、0 个 fixture，以及 5 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L16)
```python
import pytest
import transformers
from packaging import version
from transformers import AutoModel

from vllm.entrypoints.chat_utils import (
    ChatCompletionContentPartImageEmbedsParam,
    ChatCompletionContentPartImageParam,
    ChatCompletionContentPartTextParam,
)
from vllm.entrypoints.pooling.scoring.typing import ScoreMultiModalParam

from ....conftest import HfRunner, VllmRunner
```
**EN:** Imports third-party packages like `packaging.version`, `pytest`, `transformers`, project helpers such as `vllm.entrypoints.chat_utils.ChatCompletionContentPartImageEmbedsParam`, `vllm.entrypoints.chat_utils.ChatCompletionContentPartImageParam`, `vllm.entrypoints.chat_utils.ChatCompletionContentPartTextParam`.
**CN:** 导入第三方包（如 `packaging.version`、`pytest`、`transformers`）、项目内辅助模块（如 `vllm.entrypoints.chat_utils.ChatCompletionContentPartImageEmbedsParam`、`vllm.entrypoints.chat_utils.ChatCompletionContentPartImageParam`、`vllm.entrypoints.chat_utils.ChatCompletionContentPartTextParam`）。

### Module setup / 模块级配置: pytestmark, MODELS, MM_PROCESSOR_KWARGS (L18-L110)
```python
pytestmark = pytest.mark.skip(
    reason="jinaai/jina-reranker-m0 custom code is incompatible with "
    "transformers v5 (missing all_tied_weights_keys)"
)

MODELS = ["jinaai/jina-reranker-m0"]

MM_PROCESSOR_KWARGS = {
    "min_pixels": 3136,
    "max_pixels": 602112,
}

LIMIT_MM_PER_PROMPT = {"image": 2}

CHECKPOINT_TO_HF_MAPPER = {
    "visual.": "model.visual.",
# ... 71 lines omitted for brevity ...
        {"text": "数据提取么？为什么不用正则啊,你用正则不就全解决了么?"},
        {
            "image": "https://raw.githubusercontent.com/jina-ai/multimodal-reranker-test/main/handelsblatt-preview.png"
        },
    ],
}
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `pytestmark`, `MODELS`, `MM_PROCESSOR_KWARGS`, `LIMIT_MM_PER_PROMPT`, `CHECKPOINT_TO_HF_MAPPER`, `LONG_TEXT_DOC`, `TEXT_IMAGE_TEST_DATA`, `TEXT_TEXT_TEST_DATA`. Later helpers and tests reuse these values to keep scenarios concise. It also applies module-wide pytest markers.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `pytestmark`、`MODELS`、`MM_PROCESSOR_KWARGS`、`LIMIT_MM_PER_PROMPT`、`CHECKPOINT_TO_HF_MAPPER`、`LONG_TEXT_DOC`、`TEXT_IMAGE_TEST_DATA`、`TEXT_TEXT_TEST_DATA`。后续辅助函数和测试会复用这些值，以减少重复。 它还会应用模块级的 pytest 标记。

### Helper / 辅助函数: _normalize_image (L113-L119)
```python
def _normalize_image(image_val: str) -> str:
    """Normalize image value to proper format for HF model."""
    return (
        image_val
        if image_val.startswith(("http://", "https://"))
        else f"data:image/png;base64,{image_val}"
    )
```
**EN:** This helper encapsulates reusable logic in `_normalize_image`. Key inputs are `image_val`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_normalize_image` 中。 关键输入包括 `image_val`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: create_score_multimodal_param (L122-L159)
```python
def create_score_multimodal_param(
    content_parts: list[dict],
) -> list[ScoreMultiModalParam]:
    """
    Create a ScoreMultiModalParam from a list of content dictionaries.

    Each dict supports the following formats:
    - Text: {'text': 'content'}
    - Image URL: {'image': 'https://...'}
    - Image Base64: {'image': 'base64_str'}
    """
    formatted_content = []

    for part in content_parts:
        if "text" in part:
            formatted_content.append(
                ChatCompletionContentPartTextParam(
                    type="text",
# ... 12 lines omitted for brevity ...
            else:
                formatted_content.append(
                    ChatCompletionContentPartImageEmbedsParam(
                        type="image_embeds", image_embeds=image_val
                    )
                )

    return [ScoreMultiModalParam(content=[content]) for content in formatted_content]
```
**EN:** This helper encapsulates reusable logic in `create_score_multimodal_param`. Key inputs are `content_parts`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `create_score_multimodal_param` 中。 关键输入包括 `content_parts`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _run_vllm (L162-L184)
```python
def _run_vllm(
    vllm_runner: type[VllmRunner],
    model: str,
    dtype: str,
    query_strs: list[dict[str, str]],
    document_strs: list[dict[str, str]],
) -> list[float]:
    """Run vLLM reranker and return scores."""
    query = create_score_multimodal_param(query_strs)
    documents = create_score_multimodal_param(document_strs)

    with vllm_runner(
        model,
        runner="pooling",
        dtype=dtype,
        max_num_seqs=2,
        max_model_len=2048,
        mm_processor_kwargs=MM_PROCESSOR_KWARGS,
        limit_mm_per_prompt=LIMIT_MM_PER_PROMPT,
    ) as vllm_model:
        outputs = vllm_model.llm.score(query, documents)

    return [output.outputs.score for output in outputs]
```
**EN:** This helper encapsulates reusable logic in `_run_vllm`. Key inputs are `vllm_runner`, `model`, `dtype`, `query_strs`, `document_strs`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_vllm` 中。 关键输入包括 `vllm_runner`、`model`、`dtype`、`query_strs`、`document_strs`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _run_hf (L187-L231)
```python
def _run_hf(
    hf_runner: type[HfRunner],
    model: str,
    dtype: str,
    query_strs: list[dict[str, str]],
    document_strs: list[dict[str, str]],
) -> list[float]:
    """Run HuggingFace reranker and return scores."""
    query = query_strs[0]
    if "text" in query:
        query_type = "text"
        query_data = query["text"]
    elif "image" in query:
        query_type = "image"
        query_data = _normalize_image(query["image"])
    else:
        raise ValueError("Unsupported query format")

# ... 19 lines omitted for brevity ...
                score = hf_model.model.compute_score(
                    [[query_data, doc["image"]]],
                    max_length=2048,
                    query_type=query_type,
                    doc_type="image",
                )
                scores.append(score)
    return scores
```
**EN:** This helper encapsulates reusable logic in `_run_hf`. Key inputs are `hf_runner`, `model`, `dtype`, `query_strs`, `document_strs`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_hf` 中。 关键输入包括 `hf_runner`、`model`、`dtype`、`query_strs`、`document_strs`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _run_test (L234-L259)
```python
def _run_test(
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    model: str,
    dtype: str,
    query_strs: list[dict[str, str]],
    document_strs: list[dict[str, str]],
) -> None:
    """Run comparison test between vLLM and HuggingFace implementations."""
    # NOTE: take care of the order. run vLLM first, and then run HF.
    # vLLM needs a fresh new process without cuda initialization.
    # if we run HF first, the cuda initialization will be done and it
    # will hurt multiprocessing backend with fork method (the default method).

    vllm_outputs = _run_vllm(vllm_runner, model, dtype, query_strs, document_strs)
    hf_outputs = _run_hf(hf_runner, model, dtype, query_strs, document_strs)

    # Compare outputs
    assert len(hf_outputs) == len(vllm_outputs), (
        f"Output length mismatch: HF={len(hf_outputs)}, vLLM={len(vllm_outputs)}"
    )

    for i, (hf_score, vllm_score) in enumerate(zip(hf_outputs, vllm_outputs)):
        assert hf_score == pytest.approx(vllm_score, rel=0.02), (
            f"Score mismatch at index {i}: HF={hf_score}, vLLM={vllm_score}"
        )
```
**EN:** This helper encapsulates reusable logic in `_run_test`. Key inputs are `hf_runner`, `vllm_runner`, `model`, `dtype`, `query_strs`, `document_strs`. The main assertion is `len(hf_outputs) == len(vllm_outputs)` and `hf_score == pytest.approx(vllm_score, rel=0.02)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_test` 中。 关键输入包括 `hf_runner`、`vllm_runner`、`model`、`dtype`、`query_strs`、`document_strs`。 核心断言是 `len(hf_outputs) == len(vllm_outputs)` and `hf_score == pytest.approx(vllm_score, rel=0.02)`。

### Test / 测试: test_model_text_image (L262-L282)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["half"])
@pytest.mark.skipif(
    version.parse(transformers.__version__) == version.parse("4.57.5"),
    reason="Skipped for transformers==4.57.5, https://github.com/huggingface/transformers/issues/43295",
)
def test_model_text_image(
    hf_runner,
    vllm_runner,
    model: str,
    dtype: str,
) -> None:
    """Visual Documents Reranking"""
    _run_test(
        hf_runner,
        vllm_runner,
        model,
        dtype,
        TEXT_IMAGE_TEST_DATA["query"],
        TEXT_IMAGE_TEST_DATA["documents"],
    )
```
**EN:** This test validates `test_model_text_image`. It uses parameterization over `model`. Relevant pytest markers include `skipif`. Key inputs are `hf_runner`, `vllm_runner`, `model`, `dtype`.
**CN:** 这个测试验证 `test_model_text_image`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `skipif`。 关键输入包括 `hf_runner`、`vllm_runner`、`model`、`dtype`。

### Test / 测试: test_model_text_text (L285-L305)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["half"])
@pytest.mark.skipif(
    version.parse(transformers.__version__) == version.parse("4.57.5"),
    reason="Skipped for transformers==4.57.5, https://github.com/huggingface/transformers/issues/43295",
)
def test_model_text_text(
    hf_runner,
    vllm_runner,
    model: str,
    dtype: str,
) -> None:
    """Textual Documents Reranking"""
    _run_test(
        hf_runner,
        vllm_runner,
        model,
        dtype,
        TEXT_TEXT_TEST_DATA["query"],
        TEXT_TEXT_TEST_DATA["documents"],
    )
```
**EN:** This test validates `test_model_text_text`. It uses parameterization over `model`. Relevant pytest markers include `skipif`. Key inputs are `hf_runner`, `vllm_runner`, `model`, `dtype`.
**CN:** 这个测试验证 `test_model_text_text`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `skipif`。 关键输入包括 `hf_runner`、`vllm_runner`、`model`、`dtype`。

### Test / 测试: test_model_image_text (L308-L328)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["half"])
@pytest.mark.skipif(
    version.parse(transformers.__version__) == version.parse("4.57.5"),
    reason="Skipped for transformers==4.57.5, https://github.com/huggingface/transformers/issues/43295",
)
def test_model_image_text(
    hf_runner,
    vllm_runner,
    model: str,
    dtype: str,
) -> None:
    """Image Querying for Textual Documents"""
    _run_test(
        hf_runner,
        vllm_runner,
        model,
        dtype,
        IMAGE_TEXT_TEST_DATA["query"],
        IMAGE_TEXT_TEST_DATA["documents"],
    )
```
**EN:** This test validates `test_model_image_text`. It uses parameterization over `model`. Relevant pytest markers include `skipif`. Key inputs are `hf_runner`, `vllm_runner`, `model`, `dtype`.
**CN:** 这个测试验证 `test_model_image_text`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `skipif`。 关键输入包括 `hf_runner`、`vllm_runner`、`model`、`dtype`。

### Test / 测试: test_model_image_image (L331-L351)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["half"])
@pytest.mark.skipif(
    version.parse(transformers.__version__) == version.parse("4.57.5"),
    reason="Skipped for transformers==4.57.5, https://github.com/huggingface/transformers/issues/43295",
)
def test_model_image_image(
    hf_runner,
    vllm_runner,
    model: str,
    dtype: str,
) -> None:
    """Image Querying for Image Documents"""
    _run_test(
        hf_runner,
        vllm_runner,
        model,
        dtype,
        IMAGE_IMAGE_TEST_DATA["query"],
        IMAGE_IMAGE_TEST_DATA["documents"],
    )
```
**EN:** This test validates `test_model_image_image`. It uses parameterization over `model`. Relevant pytest markers include `skipif`. Key inputs are `hf_runner`, `vllm_runner`, `model`, `dtype`.
**CN:** 这个测试验证 `test_model_image_image`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `skipif`。 关键输入包括 `hf_runner`、`vllm_runner`、`model`、`dtype`。

### Test / 测试: test_model_text_mixed_documents (L354-L374)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["half"])
@pytest.mark.skipif(
    version.parse(transformers.__version__) == version.parse("4.57.5"),
    reason="Skipped for transformers==4.57.5, https://github.com/huggingface/transformers/issues/43295",
)
def test_model_text_mixed_documents(
    hf_runner,
    vllm_runner,
    model: str,
    dtype: str,
) -> None:
    """Text Query for Mixed Text and Image Documents"""
    _run_test(
        hf_runner,
        vllm_runner,
        model,
        dtype,
        TEXT_MIXED_DOCS_TEST_DATA["query"],
        TEXT_MIXED_DOCS_TEST_DATA["documents"],
    )
```
**EN:** This test validates `test_model_text_mixed_documents`. It uses parameterization over `model`. Relevant pytest markers include `skipif`. Key inputs are `hf_runner`, `vllm_runner`, `model`, `dtype`.
**CN:** 这个测试验证 `test_model_text_mixed_documents`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `skipif`。 关键输入包括 `hf_runner`、`vllm_runner`、`model`、`dtype`。

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
- **Third-party / 第三方**: `packaging.version`, `pytest`, `transformers`, `transformers.AutoModel`
- **Project / 项目内**: `vllm.entrypoints.chat_utils.ChatCompletionContentPartImageEmbedsParam`, `vllm.entrypoints.chat_utils.ChatCompletionContentPartImageParam`, `vllm.entrypoints.chat_utils.ChatCompletionContentPartTextParam`, `vllm.entrypoints.pooling.scoring.typing.ScoreMultiModalParam`
- **Local relative imports / 本地相对导入**: `....conftest.HfRunner`, `....conftest.VllmRunner`
