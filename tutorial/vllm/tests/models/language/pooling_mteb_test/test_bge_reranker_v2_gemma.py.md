# test_bge_reranker_v2_gemma.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/pooling_mteb_test/test_bge_reranker_v2_gemma.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and language-model behavior. The file defines 1 test(s), 0 fixture(s), and 4 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与语言模型行为。它定义了 1 个测试、0 个 fixture，以及 4 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L17)
```python
from typing import Any

import mteb
import numpy as np
import pytest
import torch
from torch.utils.data import DataLoader

from tests.conftest import HfRunner
from tests.models.utils import RerankModelInfo

from .mteb_score_utils import (
    MtebCrossEncoderMixin,
    mteb_test_rerank_models,
)
```
**EN:** Imports standard-library modules such as `typing.Any`, third-party packages like `mteb`, `numpy`, `pytest`, project helpers such as `tests.conftest.HfRunner`, `tests.models.utils.RerankModelInfo`, `.mteb_score_utils.MtebCrossEncoderMixin`.
**CN:** 导入标准库模块（如 `typing.Any`）、第三方包（如 `mteb`、`numpy`、`pytest`）、项目内辅助模块（如 `tests.conftest.HfRunner`、`tests.models.utils.RerankModelInfo`、`.mteb_score_utils.MtebCrossEncoderMixin`）。

### Module setup / 模块级配置: RERANK_MODELS, PROMPT (L19-L37)
```python
RERANK_MODELS = [
    RerankModelInfo(
        "BAAI/bge-reranker-v2-gemma",
        architecture="GemmaForSequenceClassification",
        hf_overrides={
            "architectures": ["GemmaForSequenceClassification"],
            "classifier_from_token": ["Yes"],
            "method": "no_post_processing",
        },
        mteb_score=0.33757,
        seq_pooling_type="LAST",
        attn_type="decoder",
        is_prefix_caching_supported=True,
        is_chunked_prefill_supported=True,
        chat_template_name="bge-reranker-v2-gemma.jinja",
    ),
]

PROMPT = "Given a query A and a passage B, determine whether the passage contains an answer to the query by providing a prediction of either 'Yes' or 'No'."  # noqa: E501
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `RERANK_MODELS`, `PROMPT`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `RERANK_MODELS`、`PROMPT`。后续辅助函数和测试会复用这些值，以减少重复。

### Class / 类: GemmaRerankerHfRunner (L40-L127)
```python
class GemmaRerankerHfRunner(MtebCrossEncoderMixin, HfRunner):
    def __init__(
        self, model_name: str, dtype: str = "auto", *args: Any, **kwargs: Any
    ) -> None:
        from transformers import AutoModelForCausalLM, AutoTokenizer

        HfRunner.__init__(
            self,
            model_name=model_name,
            auto_cls=AutoModelForCausalLM,
            dtype=dtype,
            **kwargs,
        )

# ... 66 lines omitted for brevity ...
                .view(
                    -1,
                )
                .float()
                .sigmoid()
            )
            scores.append(_scores[0].item())
        return torch.Tensor(scores)
```
**EN:** This class groups related scenarios in `GemmaRerankerHfRunner`. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `__init__`, `predict`.
**CN:** 该类将与 `GemmaRerankerHfRunner` 相关的场景组织在一起。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `__init__`、`predict`。

### Helper method / 辅助方法: GemmaRerankerHfRunner.__init__ (L41-L55)
```python
    def __init__(
        self, model_name: str, dtype: str = "auto", *args: Any, **kwargs: Any
    ) -> None:
        from transformers import AutoModelForCausalLM, AutoTokenizer

        HfRunner.__init__(
            self,
            model_name=model_name,
            auto_cls=AutoModelForCausalLM,
            dtype=dtype,
            **kwargs,
        )

        self.tokenizer = AutoTokenizer.from_pretrained(model_name, padding_side="left")
        self.yes_loc = self.tokenizer.convert_tokens_to_ids("Yes")
```
**EN:** This helper encapsulates reusable logic in `GemmaRerankerHfRunner.__init__`. Key inputs are `model_name`, `dtype`.
**CN:** 这个辅助函数将可复用逻辑封装在 `GemmaRerankerHfRunner.__init__` 中。 关键输入包括 `model_name`、`dtype`。

### Helper method / 辅助方法: GemmaRerankerHfRunner.predict (L57-L127)
```python
    @torch.no_grad
    def predict(
        self,
        inputs1: DataLoader[mteb.types.BatchedInput],
        inputs2: DataLoader[mteb.types.BatchedInput],
        *args,
        **kwargs,
    ) -> np.ndarray:
        queries = [text for batch in inputs1 for text in batch["text"]]
        corpus = [text for batch in inputs2 for text in batch["text"]]

        def get_inputs(pairs, tokenizer, prompt=None):
            if prompt is None:
                prompt = PROMPT

            sep = "\n"
            prompt_inputs = tokenizer(
                prompt, return_tensors=None, add_special_tokens=False
# ... 45 lines omitted for brevity ...
                .view(
                    -1,
                )
                .float()
                .sigmoid()
            )
            scores.append(_scores[0].item())
        return torch.Tensor(scores)
```
**EN:** This helper encapsulates reusable logic in `GemmaRerankerHfRunner.predict`. Key inputs are `inputs1`, `inputs2`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `GemmaRerankerHfRunner.predict` 中。 关键输入包括 `inputs1`、`inputs2`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_rerank_models_mteb (L130-L136)
```python
@pytest.mark.parametrize("model_info", RERANK_MODELS)
def test_rerank_models_mteb(vllm_runner, model_info: RerankModelInfo) -> None:
    mteb_test_rerank_models(
        vllm_runner,
        model_info,
        hf_runner=GemmaRerankerHfRunner,
    )
```
**EN:** This test validates `test_rerank_models_mteb`. It uses parameterization over `model_info`. Key inputs are `vllm_runner`, `model_info`.
**CN:** 这个测试验证 `test_rerank_models_mteb`。 它通过参数化组合 `model_info`。 关键输入包括 `vllm_runner`、`model_info`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `typing.Any`
- **Third-party / 第三方**: `mteb`, `numpy`, `pytest`, `torch`, `torch.utils.data.DataLoader`, `transformers.AutoModelForCausalLM`, `transformers.AutoTokenizer`
- **Project / 项目内**: `tests.conftest.HfRunner`, `tests.models.utils.RerankModelInfo`
- **Local relative imports / 本地相对导入**: `.mteb_score_utils.MtebCrossEncoderMixin`, `.mteb_score_utils.mteb_test_rerank_models`
