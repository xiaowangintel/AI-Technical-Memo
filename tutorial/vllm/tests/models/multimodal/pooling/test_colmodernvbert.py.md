# test_colmodernvbert.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/pooling/test_colmodernvbert.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and multimodal processing. The file defines 4 test(s), 0 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与多模态处理。它定义了 4 个测试、0 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L10-L13)
```python
import pytest
import torch

from vllm.entrypoints.pooling.scoring.utils import compute_maxsim_score
```
**EN:** Imports third-party packages like `pytest`, `torch`, project helpers such as `vllm.entrypoints.pooling.scoring.utils.compute_maxsim_score`.
**CN:** 导入第三方包（如 `pytest`、`torch`）、项目内辅助模块（如 `vllm.entrypoints.pooling.scoring.utils.compute_maxsim_score`）。

### Module setup / 模块级配置: MODEL_NAME, COLBERT_DIM, DTYPE (L15-L17)
```python
MODEL_NAME = "ModernVBERT/colmodernvbert-merged"
COLBERT_DIM = 128
DTYPE = "half"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `COLBERT_DIM`, `DTYPE`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`COLBERT_DIM`、`DTYPE`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_colmodernvbert_text_token_embed (L25-L39)
```python
def test_colmodernvbert_text_token_embed(vllm_runner):
    """Text query produces per-token embeddings with shape (seq_len, 128)."""
    with vllm_runner(
        MODEL_NAME,
        runner="pooling",
        dtype=DTYPE,
        enforce_eager=True,
    ) as vllm_model:
        outputs = vllm_model.token_embed(["What is machine learning?"])

        assert len(outputs) == 1
        emb = torch.tensor(outputs[0])
        assert emb.dim() == 2
        assert emb.shape[1] == COLBERT_DIM
        assert emb.shape[0] > 1
```
**EN:** This test validates `test_colmodernvbert_text_token_embed`. Key inputs are `vllm_runner`. The main assertion is `len(outputs) == 1` and `emb.dim() == 2`.
**CN:** 这个测试验证 `test_colmodernvbert_text_token_embed`。 关键输入包括 `vllm_runner`。 核心断言是 `len(outputs) == 1` and `emb.dim() == 2`。

### Test / 测试: test_colmodernvbert_text_relevance_ordering (L42-L59)
```python
def test_colmodernvbert_text_relevance_ordering(vllm_runner):
    """Relevant documents score higher than irrelevant ones."""
    query = "What is machine learning?"
    documents = [
        "Machine learning is a subset of artificial intelligence.",
        "The weather in Paris is mild in spring.",
    ]

    with vllm_runner(
        MODEL_NAME,
        runner="pooling",
        dtype=DTYPE,
        enforce_eager=True,
    ) as vllm_model:
        scores = vllm_model.score(query, documents)

        assert len(scores) == 2
        assert scores[0] > scores[1], "ML doc should score higher than weather doc"
```
**EN:** This test validates `test_colmodernvbert_text_relevance_ordering`. Key inputs are `vllm_runner`. The main assertion is `len(scores) == 2` and `scores[0] > scores[1]`.
**CN:** 这个测试验证 `test_colmodernvbert_text_relevance_ordering`。 关键输入包括 `vllm_runner`。 核心断言是 `len(scores) == 2` and `scores[0] > scores[1]`。

### Test / 测试: test_colmodernvbert_text_late_interaction (L62-L83)
```python
def test_colmodernvbert_text_late_interaction(vllm_runner):
    """MaxSim scoring via vLLM matches manual computation."""
    query = "What is the capital of France?"
    doc = "The capital of France is Paris."

    with vllm_runner(
        MODEL_NAME,
        runner="pooling",
        dtype=DTYPE,
        enforce_eager=True,
    ) as vllm_model:
        q_out = vllm_model.token_embed([query])
        d_out = vllm_model.token_embed([doc])

        q_emb = torch.tensor(q_out[0])
        d_emb = torch.tensor(d_out[0])
        manual_score = compute_maxsim_score(q_emb, d_emb).item()

        vllm_scores = vllm_model.score(query, doc)

        assert len(vllm_scores) == 1
        assert vllm_scores[0] == pytest.approx(manual_score, rel=0.01)
```
**EN:** This test validates `test_colmodernvbert_text_late_interaction`. Key inputs are `vllm_runner`. The main assertion is `len(vllm_scores) == 1` and `vllm_scores[0] == pytest.approx(manual_score, rel=0.01)`.
**CN:** 这个测试验证 `test_colmodernvbert_text_late_interaction`。 关键输入包括 `vllm_runner`。 核心断言是 `len(vllm_scores) == 1` and `vllm_scores[0] == pytest.approx(manual_score, rel=0.01)`。

### Test / 测试: test_colmodernvbert_image_token_embed (L91-L115)
```python
def test_colmodernvbert_image_token_embed(vllm_runner, image_assets):
    """Image input produces per-token embeddings including vision tokens."""
    with vllm_runner(
        MODEL_NAME,
        runner="pooling",
        dtype=DTYPE,
        enforce_eager=True,
    ) as vllm_model:
        image = image_assets[0].pil_image
        inputs = vllm_model.get_inputs(
            [""],
            images=[image],
        )
        req_outputs = vllm_model.llm.encode(
            inputs,
            pooling_task="token_embed",
        )
        outputs = [req_output.outputs.data for req_output in req_outputs]

        assert len(outputs) == 1
        emb = torch.tensor(outputs[0])
        assert emb.dim() == 2
        assert emb.shape[1] == COLBERT_DIM
        # Should have at least the image tokens (64 after pixel shuffle)
        assert emb.shape[0] >= 64
```
**EN:** This test validates `test_colmodernvbert_image_token_embed`. Key inputs are `vllm_runner`, `image_assets`. The main assertion is `len(outputs) == 1` and `emb.dim() == 2`.
**CN:** 这个测试验证 `test_colmodernvbert_image_token_embed`。 关键输入包括 `vllm_runner`、`image_assets`。 核心断言是 `len(outputs) == 1` and `emb.dim() == 2`。

## Key Concepts / 关键概念
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `pytest`, `torch`
- **Project / 项目内**: `vllm.entrypoints.pooling.scoring.utils.compute_maxsim_score`
