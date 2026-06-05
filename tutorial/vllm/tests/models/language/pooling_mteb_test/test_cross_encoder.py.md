# test_cross_encoder.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/pooling_mteb_test/test_cross_encoder.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and language-model behavior. The file defines 1 test(s), 0 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与语言模型行为。它定义了 1 个测试、0 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L9)
```python
import pytest

from tests.models.utils import (
    RerankModelInfo,
)

from .mteb_score_utils import mteb_test_rerank_models
```
**EN:** Imports third-party packages like `pytest`, project helpers such as `tests.models.utils.RerankModelInfo`, `.mteb_score_utils.mteb_test_rerank_models`.
**CN:** 导入第三方包（如 `pytest`）、项目内辅助模块（如 `tests.models.utils.RerankModelInfo`、`.mteb_score_utils.mteb_test_rerank_models`）。

### Module setup / 模块级配置: RERANK_MODELS (L11-L31)
```python
RERANK_MODELS = [
    RerankModelInfo(
        "cross-encoder/ms-marco-TinyBERT-L-2-v2",
        architecture="BertForSequenceClassification",
        seq_pooling_type="CLS",
        attn_type="encoder_only",
        is_prefix_caching_supported=False,
        is_chunked_prefill_supported=False,
        mteb_score=0.32898,
    ),
    RerankModelInfo(
        "tomaarsen/Qwen3-Reranker-0.6B-seq-cls",
        architecture="Qwen3ForSequenceClassification",
        seq_pooling_type="LAST",
        attn_type="decoder",
        is_prefix_caching_supported=True,
        is_chunked_prefill_supported=True,
        chat_template_name="qwen3_reranker.jinja",
        mteb_score=0.33459,
    ),
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `RERANK_MODELS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `RERANK_MODELS`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_rerank_models_mteb (L34-L36)
```python
@pytest.mark.parametrize("model_info", RERANK_MODELS)
def test_rerank_models_mteb(vllm_runner, model_info: RerankModelInfo) -> None:
    mteb_test_rerank_models(vllm_runner, model_info)
```
**EN:** This test validates `test_rerank_models_mteb`. It uses parameterization over `model_info`. Key inputs are `vllm_runner`, `model_info`.
**CN:** 这个测试验证 `test_rerank_models_mteb`。 它通过参数化组合 `model_info`。 关键输入包括 `vllm_runner`、`model_info`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `tests.models.utils.RerankModelInfo`
- **Local relative imports / 本地相对导入**: `.mteb_score_utils.mteb_test_rerank_models`
