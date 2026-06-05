# test_qwen3_reranker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/pooling_mteb_test/test_qwen3_reranker.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and language-model behavior. The file defines 2 test(s), 0 fixture(s), and 4 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与语言模型行为。它定义了 2 个测试、0 个 fixture，以及 4 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L16)
```python
from typing import Any

import mteb
import numpy as np
import pytest
import torch
from torch.utils.data import DataLoader

from tests.conftest import HfRunner
from tests.models.utils import RerankModelInfo
from tests.utils import multi_gpu_test

from .mteb_score_utils import MtebCrossEncoderMixin, mteb_test_rerank_models
```
**EN:** Imports standard-library modules such as `typing.Any`, third-party packages like `mteb`, `numpy`, `pytest`, project helpers such as `tests.conftest.HfRunner`, `tests.models.utils.RerankModelInfo`, `tests.utils.multi_gpu_test`.
**CN:** 导入标准库模块（如 `typing.Any`）、第三方包（如 `mteb`、`numpy`、`pytest`）、项目内辅助模块（如 `tests.conftest.HfRunner`、`tests.models.utils.RerankModelInfo`、`tests.utils.multi_gpu_test`）。

### Module setup / 模块级配置: qwen3_reranker_hf_overrides, RERANK_MODELS (L18-L44)
```python
qwen3_reranker_hf_overrides = {
    "architectures": ["Qwen3ForSequenceClassification"],
    "classifier_from_token": ["no", "yes"],
    "is_original_qwen3_reranker": True,
}

RERANK_MODELS = [
    RerankModelInfo(
        "Qwen/Qwen3-Reranker-0.6B",
        architecture="Qwen3ForSequenceClassification",
        hf_overrides=qwen3_reranker_hf_overrides,
        chat_template_name="qwen3_reranker.jinja",
        seq_pooling_type="LAST",
        attn_type="decoder",
        is_prefix_caching_supported=True,
        is_chunked_prefill_supported=True,
# ... 5 lines omitted for brevity ...
        architecture="Qwen3ForSequenceClassification",
        chat_template_name="qwen3_reranker.jinja",
        hf_overrides=qwen3_reranker_hf_overrides,
        enable_test=False,
    ),
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `qwen3_reranker_hf_overrides`, `RERANK_MODELS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `qwen3_reranker_hf_overrides`、`RERANK_MODELS`。后续辅助函数和测试会复用这些值，以减少重复。

### Class / 类: Qwen3RerankerHfRunner (L47-L108)
```python
class Qwen3RerankerHfRunner(MtebCrossEncoderMixin, HfRunner):
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

# ... 40 lines omitted for brevity ...

        scores = []
        for prompt in prompts:
            inputs = tokenizer([prompt], return_tensors="pt")
            inputs = self.wrap_device(inputs)
            score = compute_logits(inputs)
            scores.append(score[0].item())
        return torch.Tensor(scores)
```
**EN:** This class groups related scenarios in `Qwen3RerankerHfRunner`. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `__init__`, `predict`.
**CN:** 该类将与 `Qwen3RerankerHfRunner` 相关的场景组织在一起。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `__init__`、`predict`。

### Helper method / 辅助方法: Qwen3RerankerHfRunner.__init__ (L48-L64)
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
        self.token_false_id = self.tokenizer.convert_tokens_to_ids("no")
        self.token_true_id = self.tokenizer.convert_tokens_to_ids("yes")
        self.max_length = 40960
```
**EN:** This helper encapsulates reusable logic in `Qwen3RerankerHfRunner.__init__`. Key inputs are `model_name`, `dtype`.
**CN:** 这个辅助函数将可复用逻辑封装在 `Qwen3RerankerHfRunner.__init__` 中。 关键输入包括 `model_name`、`dtype`。

### Helper method / 辅助方法: Qwen3RerankerHfRunner.predict (L66-L108)
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

        tokenizer = self.tokenizer
        prompts = []
        for query, document in zip(queries, corpus):
            conversation = [
                {"role": "query", "content": query},
                {"role": "document", "content": document},
            ]
# ... 17 lines omitted for brevity ...

        scores = []
        for prompt in prompts:
            inputs = tokenizer([prompt], return_tensors="pt")
            inputs = self.wrap_device(inputs)
            score = compute_logits(inputs)
            scores.append(score[0].item())
        return torch.Tensor(scores)
```
**EN:** This helper encapsulates reusable logic in `Qwen3RerankerHfRunner.predict`. Key inputs are `inputs1`, `inputs2`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `Qwen3RerankerHfRunner.predict` 中。 关键输入包括 `inputs1`、`inputs2`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_rerank_models_mteb (L111-L113)
```python
@pytest.mark.parametrize("model_info", RERANK_MODELS)
def test_rerank_models_mteb(vllm_runner, model_info: RerankModelInfo) -> None:
    mteb_test_rerank_models(vllm_runner, model_info, hf_runner=Qwen3RerankerHfRunner)
```
**EN:** This test validates `test_rerank_models_mteb`. It uses parameterization over `model_info`. Key inputs are `vllm_runner`, `model_info`.
**CN:** 这个测试验证 `test_rerank_models_mteb`。 它通过参数化组合 `model_info`。 关键输入包括 `vllm_runner`、`model_info`。

### Test / 测试: test_rerank_models_mteb_tp (L116-L130)
```python
@pytest.mark.parametrize("model_info", RERANK_MODELS)
@multi_gpu_test(num_gpus=2)
def test_rerank_models_mteb_tp(vllm_runner, model_info: RerankModelInfo) -> None:
    assert model_info.architecture == "Qwen3ForSequenceClassification"

    vllm_extra_kwargs: dict[str, Any] = {
        "tensor_parallel_size": 2,
    }

    mteb_test_rerank_models(
        vllm_runner,
        model_info,
        vllm_extra_kwargs=vllm_extra_kwargs,
        hf_runner=Qwen3RerankerHfRunner,
    )
```
**EN:** This test validates `test_rerank_models_mteb_tp`. It uses parameterization over `model_info`. Key inputs are `vllm_runner`, `model_info`. The main assertion is `model_info.architecture == 'Qwen3ForSequenceClassification'`.
**CN:** 这个测试验证 `test_rerank_models_mteb_tp`。 它通过参数化组合 `model_info`。 关键输入包括 `vllm_runner`、`model_info`。 核心断言是 `model_info.architecture == 'Qwen3ForSequenceClassification'`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `typing.Any`
- **Third-party / 第三方**: `mteb`, `numpy`, `pytest`, `torch`, `torch.utils.data.DataLoader`, `transformers.AutoModelForCausalLM`, `transformers.AutoTokenizer`
- **Project / 项目内**: `tests.conftest.HfRunner`, `tests.models.utils.RerankModelInfo`, `tests.utils.multi_gpu_test`
- **Local relative imports / 本地相对导入**: `.mteb_score_utils.MtebCrossEncoderMixin`, `.mteb_score_utils.mteb_test_rerank_models`
