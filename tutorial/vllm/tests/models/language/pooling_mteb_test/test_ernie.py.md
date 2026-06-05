# test_ernie.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/pooling_mteb_test/test_ernie.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and language-model behavior. The file defines 2 test(s), 0 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与语言模型行为。它定义了 2 个测试、0 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L9)
```python
import pytest

from tests.models.language.pooling.embed_utils import correctness_test_embed_models
from tests.models.utils import EmbedModelInfo

from .mteb_embed_utils import mteb_test_embed_models
```
**EN:** Imports third-party packages like `pytest`, project helpers such as `tests.models.language.pooling.embed_utils.correctness_test_embed_models`, `tests.models.utils.EmbedModelInfo`, `.mteb_embed_utils.mteb_test_embed_models`.
**CN:** 导入第三方包（如 `pytest`）、项目内辅助模块（如 `tests.models.language.pooling.embed_utils.correctness_test_embed_models`、`tests.models.utils.EmbedModelInfo`、`.mteb_embed_utils.mteb_test_embed_models`）。

### Module setup / 模块级配置: MODELS (L11-L22)
```python
MODELS = [
    EmbedModelInfo(
        "shibing624/text2vec-base-chinese-sentence",
        architecture="ErnieModel",
        mteb_score=0.536523112,
        seq_pooling_type="MEAN",
        attn_type="encoder_only",
        is_prefix_caching_supported=False,
        is_chunked_prefill_supported=False,
        enable_test=True,
    ),
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODELS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODELS`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_embed_models_mteb (L25-L32)
```python
@pytest.mark.parametrize("model_info", MODELS)
def test_embed_models_mteb(hf_runner, vllm_runner, model_info: EmbedModelInfo) -> None:
    mteb_test_embed_models(
        hf_runner,
        vllm_runner,
        model_info,
        vllm_extra_kwargs={"gpu_memory_utilization": 0.2},
    )
```
**EN:** This test validates `test_embed_models_mteb`. It uses parameterization over `model_info`. Key inputs are `hf_runner`, `vllm_runner`, `model_info`.
**CN:** 这个测试验证 `test_embed_models_mteb`。 它通过参数化组合 `model_info`。 关键输入包括 `hf_runner`、`vllm_runner`、`model_info`。

### Test / 测试: test_embed_models_correctness (L35-L45)
```python
@pytest.mark.parametrize("model_info", MODELS)
def test_embed_models_correctness(
    hf_runner, vllm_runner, model_info: EmbedModelInfo, example_prompts
) -> None:
    correctness_test_embed_models(
        hf_runner,
        vllm_runner,
        model_info,
        example_prompts,
        vllm_extra_kwargs={"gpu_memory_utilization": 0.2},
    )
```
**EN:** This test validates `test_embed_models_correctness`. It uses parameterization over `model_info`. Key inputs are `hf_runner`, `vllm_runner`, `model_info`, `example_prompts`.
**CN:** 这个测试验证 `test_embed_models_correctness`。 它通过参数化组合 `model_info`。 关键输入包括 `hf_runner`、`vllm_runner`、`model_info`、`example_prompts`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `tests.models.language.pooling.embed_utils.correctness_test_embed_models`, `tests.models.utils.EmbedModelInfo`
- **Local relative imports / 本地相对导入**: `.mteb_embed_utils.mteb_test_embed_models`
